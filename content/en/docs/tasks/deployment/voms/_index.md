---
title: "Deploying the IAM VOMS attribute authority"
weight: 120
description: >
  Instructions on how to deploy and configure the IAM VOMS attribute authority micro-service
---

## The IAM VOMS attribute authority

The IAM [VOMS attribute authority][voms-aa] (AA) provide a backward-compatible
[VOMS][voms] support for a Virtual Organization managed with IAM.

### Deployment architecture

![VOMS AA deployment](../img/iam-voms-aa.png)

The VOMS attribute authority can access the IAM database and encode IAM groups
and other attributes in a standard VOMS attribute certificate. This means in
practice that IAM can act both as an OAuth/OpenID Connect authorization server
and as a VOMS server for a given organization. TLS termination and client VOMS
atttribute certificate parsing and validation is delegated to an [NGINX VOMS module][nginx-httpg-voms],
which can de deployed as a sidecar service that just protects the
VOMS AA or also in front of the IAM backend application. The two scenarios are
depicted above.

In order to deploy a VOMS attribute authority, you are recommended to use the following
docker images:

- `cnafsd/nginx-httpg-voms`, for a NGINX server configured with the [VOMS module][nginx-httpg-voms],
which replaces [OpenResty VOMS][openresty-voms]
- `indigoiam/voms-aa-bp:{{< param voms_aa_version >}}`, for the VOMS AA service

Deployment from packages is not currently supported for the VOMS attribute
authority.

### NGINX VOMS module configuration

The NGINX VOMS module requires:

- the IGTF trust anchors properly configured; see the [EGI trust anchors][egi-trustanchors]
container.
- The certificate and private key to use: the certificate issuer must be one of
the CA present in the EGI trust anchors. See the specific section below.
- the `vomsdir` folder, with the VOMS LSC configuration generated starting from
  the VOMS attribute authority X.509 credential.

To generate the LSC configuration, you need to get the subject and issuer of the
VOMS AA X.509 credential and put them in a file named as the fully qualified
domain name of the VOMS attribute authority with the `.lsc` extension.

Let's assume that the IAM VOMS AA will answer on `voms.test.example` for the VO
`example.vo`.  The following command will generate the appropriate LSC file
(`X509_VOMS_DIR` can be set to any directory where you have writing privileges):

```console
> mkdir -p ${X509_VOMS_DIR}/example.vo
> openssl x509 -in voms_local_io.cert.pem -noout -subject -issuer -nameopt compat | \
  gsed -e 's/^subject=//' -e 's/^issuer=//' > \
  ${X509_VOMS_DIR}/example.vo/voms.test.example.lsc
```

For an example NGINX configuration, see the one use in [VOMS-enabled NGINX container][nginx-httpd-voms-config].

#### VOMS AA configuration

The VOMS AA is a Spring Boot application that shares the persistence
layer implementation with IAM, and as such can inspect the IAM database. It can share
the same environment file as the login service it is associated with, with the addition of the
following variables:

- `VOMS_AA_HOST`: host name used to access the VOMS AA service
- `VOMS_AA_PORT` VOMS AA port
- `VOMS_AA_V0`: VO served by the VOMS AA service

You also need to create a VOMS AA YAML configuration file with the content below (its content should
be generic as it relies on standard environment variables used to configure the login service) and map it
to the container as `/workspace/config/application.yml`.

```yaml
server:
  address: 0.0.0.0
  port: 8080
  use-forward-headers: true
  max-http-header-size: 16000

spring:
  main:
    banner-mode: "off"

  jpa:
    open-in-view: true

  datasource:
    dataSourceClassName: com.mysql.jdbc.jdbc2.optional.MysqlDataSource
    url: jdbc:mysql://${IAM_DB_HOST}:${IAM_DB_PORT:3306}/${IAM_DB_NAME}?useLegacyDatetimeCode=false&serverTimezone=UTC&useSSL=false
    username: ${IAM_DB_USERNAME}
    password: ${IAM_DB_PASSWORD}
    max-active: ${IAM_DB_MAX_ACTIVE:10}
    max-idle:  ${IAM_DB_MAX_IDLE:5}
    initial-size: ${IAM_DB_INITIAL_SIZE:1}
    test-while-idle: ${IAM_DB_TEST_WHILE_IDLE:true}
    test-on-borrow: ${IAM_DB_TEST_ON_BORROW:true}
    validation-query: ${IAM_DB_VALIDATION_QUERY:SELECT 1}
    time-between-eviction-runs-millis: ${IAM_DB_TIME_BETWEEN_EVICTION_RUNS_MILLIS:5000}
    min-evictable-idle-time-millis: ${IAM_DB_MIN_EVICTABLE_IDLE_TIME_MILLIS:60000}

  flyway:
    enabled: false

voms:
  tls:
    certificate-path: /certs/hostcert.pem
    private-key-path: /certs/hostkey.pem
    trust-anchors-dir: /etc/grid-security/certificates
    trust-anchors-refresh-interval-secs: 14400
  aa:
    host: ${VOMS_AA_HOST}
    port: ${VOMS_AA_PORT}
    vo-name: ${VOMS_AA_VO}
    use-legacy-fqan-encoding: ${VOMS_AA_USE_LEGACY_FQAN_ENCODING:true}
```

To map this file (in the example below, `voms-aa.yml`as `/workspace/config/application.yml`, you
typically need to add the following volume definition the VOMS AA section of your Podman/Docker
compose file:

```
        volumes:
            - /path/to/voms-aa.yml:/workspace/config/application.yml:ro,Z
```

*Note: `Z` at the end of the line is required only if you use SELinux*

In addition, you need to configure the EGI trust anchors and the VOMS AA certificate and
private key, as described below.

### Configuration of EGI trust anchors and VOMS AA certificate

Both the VOMS-enabled NGINX service and the VOMS AA service require the proper configuration
of the container the certificate and private key to use and the EGI trust anchors. If using
Podman/Docker Compose, itt is typically done with:

```
        volumes:
            - /path/to/certs/voms.test.example.cert.pem:/certs/hostcert.pem:ro,Z
            - /path/to/certs/voms.test.example.cert.key:/certs/hostkey.pem:ro,Z
            - /etc/grid-security/certificates:/etc/grid-security/certificates:ro,Z
```

The line to map EGI trust anchor may be different if you use [EGI trust anchors][egi-trustanchors]
container. See the Podman/Docker Compose example for more information.

Note that the private key must be readable by the user running the service. As it is not a good
practice to make it world readable, it may be necessary to adjust its owner.

### Podman/Docker Compose example

For an example of the container configurations, see the [VOMS AA Docker compose
file][voms-aa-compose].

[openresty-voms]: https://baltig.infn.it/cnafsd/ngx_http_voms_module
[voms-aa]: https://github.com/indigo-iam/iam/tree/master/iam-voms-aa
[nginx-httpd-voms-config]: https://github.com/indigo-iam/iam/blob/master/compose/voms-deploy/assets/nginx/conf.d/voms.test.example.conf
[voms-aa-compose]: https://github.com/indigo-iam/iam/blob/master/compose/voms-deploy/docker-compose.yml
[voms]: http://italiangrid.github.io/voms/
[egi-trustanchors]: https://github.com/indigo-iam/egi-trust-anchors-container/
[nginx-httpg-voms]: https://baltig.infn.it/cnafsd/ngx_http_voms_module