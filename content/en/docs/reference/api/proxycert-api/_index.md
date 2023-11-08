---
title: "Proxy certificate API"
linkTitle: "Proxy certificate API"
weight: 1
---

IAM provides a RESTful API that can be used to download the X.509 proxy certificate previously uploaded in IAM dashboard by the user. \
This is a plain proxy obtained by running `voms-proxy-init` command.

The API has been added mainly for the integration of IAM with the [RCAuth.eu][RCauth] online certificate authority. The X.509 proxy certificate can be uploaded using the [IAM Account Proxy Certificate API](../../api/account-api/#proxy-certificate).

The `proxy:generate` OAuth scope is required to access this API. \
Note that this scope is restricted in the default IAM configuration,
i.e. can be assigned to clients only by IAM administrators.

## POST `/iam/proxycert`

Returns a JSON representation of the proxy certificate of the user.

```bash
curl -s -X POST -H "Authorization: Bearer ${BT}" \
  -d client_id=${CLIENT_ID} \
  -d client_secret=${CLIENT_SECRET} \
  -d lifetimeSecs=${PROXY_CERT_LIFETIME_SECS} \
  http://localhost:8080/iam/proxycert
```

The user can request a specific lifetime of the proxy
that is less than or equal to the maximum lifetime of the proxy itself
and also limited by IAM configuration options `max-ac-lifetime-in-seconds`
(as shown in [VOMS AA configuration](../../../../docs/tasks/deployment/voms/#voms-aa-configuration)).

IAM response:

```json
{
  "subject": "CN=2050153474,CN=813462308,CN=John Doe jdoe@infn.it,O=Istituto Nazionale di Fisica Nucleare,C=IT,DC=tcs,DC=terena,DC=org",
  "issuer": "CN=813462308,CN=John Doe jdoe@infn.it,O=Istituto Nazionale di Fisica Nucleare,C=IT,DC=tcs,DC=terena,DC=org",
  "identity": "CN=John Doe jdoe@infn.it,O=Istituto Nazionale di Fisica Nucleare,C=IT,DC=tcs,DC=terena,DC=org",
  "not_after": "1663991299000",
  "certificate_chain": "-----BEGIN CERTIFICATE-----\nMI...lCU=\n-----END CERTIFICATE-----\n"
}
```

N.B. The first proxy certificate found is downloaded.

If one account has two or more associated proxy certificates, the issuerDn (e.g. `CN=Test CA,O=IGI,C=IT`) should be specified in the request to download a specific one:

```bash
curl -s -X POST -H "Authorization: Bearer ${BT}" \
  -d client_id=${CLIENT_ID} \
  -d client_secret=${CLIENT_SECRET} \
  -d lifetimeSecs=${PROXY_CERT_LIFETIME_SECS} \
  -d issuerDn=${ISSUER_DN} \
  http://localhost:8080/iam/proxycert
```

[RCauth]: http://rcauth.eu/
