---
title: "OpenID-Connect authentication"
linkTitle: "OpenID-Connect authentication"
weight: 1
---

IAM supports brokered OpenID Connect authentication to external identity
providers (e.g., Google). 

When brokered OpenID Connect authentication is turned on a IAM instance, users
can log into the service delegating authentication to one of the supported
OpenID Connect providers (OP).

To enable brokered OpenID Connect authentication on your IAM instance you have to:

1. Obtain client credentials and other configuration details for each of the
  external OpenID Connect providers;
2. Provide the OIDC configuration file defining the configuration of the supported OPs;
3. Enable the `oidc` profile.

## Obtain OP configuration and client credentials

In order to act as a relying party (RP) for an external OpenID Connect provider (OP), IAM
will need at least:

- client credentials (a clientId and clientSecret);
- an [OpenID Connect provider discovery endpoint URL][oidc-discovery-url];

As an example, see the the [Google identity platform OpenID Connect
guide][google-oidc] for help on this.

## Provide the OIDC configuration 

### Google configuration

If Google is the only OpenID Connect provider you want to support, the configuration
embedded in the IAM war file already provides what you need, and you just need to:

- Enable the `oidc` profile;
- Pass the google client credentials to IAM via the following environment
  variables:

  - `IAM_GOOGLE_CLIENT_ID` 
  - `IAM_GOOGLE_CLIENT_SECRET`


### Multiple providers configuration

To configure multiple providers, or to override the default configuration, you must 
provide an `application-oidc.yaml` configuration file and make it available to IAM.

Trusted OPs are configured for IAM via the `application-oidc.yaml` configuration file.

This is a Spring Boot configuration file that is activated when the `oidc` profile is turned on.

The default configuration file, embedded in the IAM war file, supports the
configuration of a single provider, Google, for backward compatibility, and looks
like this:

```yaml
oidc:
  providers:
  - name: google
    issuer: https://accounts.google.com
    client:
      clientId: ${IAM_GOOGLE_CLIENT_ID}
      clientSecret: ${IAM_GOOGLE_CLIENT_SECRET}
      redirectUris: ${iam.baseUrl}/openid_connect_login
      scope: openid,profile,email,address,phone
    loginButton:
      text: Google
      style: btn-social btn-google
      image:
        fa-icon: google
```

This configuration allows to configure support for Google by:

- activating the `oidc` profile;
- Setting the `IAM_GOOGLE_CLIENT_ID` and `IAM_GOOGLE_CLIENT_SECRET` environment
  variables to provide the Google client credentials;

To enable multiple providers, you need to override the default oidc configuration.

```yaml
oidc:
  providers:
  - name: google
    issuer: https://accounts.google.com
    client:
      clientId: ${IAM_GOOGLE_CLIENT_ID}
      clientSecret: ${IAM_GOOGLE_CLIENT_SECRET}
      redirectUris: ${iam.baseUrl}/openid_connect_login
      scope: openid,profile,email,address,phone
    loginButton:
      text: Google
      style: btn-social btn-google
      image:
        fa-icon: google
  - name: egi-checkin
    issuer: ${IAM_CHECKIN_ISSUER}
    client:
      clientId: ${IAM_CHECKIN_CLIENT_ID}
      clientSecret: ${IAM_CHECKIN_CLIENT_SECRET}
      redirectUris: ${iam.baseUrl}/openid_connect_login
      scope: ${IAM_CHECKIN_CLIENT_SCOPE:openid,profile,email}
    loginButton:
      text: EGI CheckIn
      title: Sign in with EGI CheckIn
      image:
        url: https://egi.eu/wp-content/uploads/2016/05/cropped-logo_site-1-300x300.png
        size: medium
```

The example configuration above shows how to configure support for Google and
the `EGI CheckIn` provider, with a custom button configuration for the CheckIn
provider.

See the [configuration reference][conf-ref] for instructions on how to override
the default IAM configuration.

## Registration form: filling information from IdP

See [registration configuration][external-idp-config].

[external-idp-config]: {{< ref "/docs/reference/configuration/registration/#registration-with-external-idp" >}}
[google-oidc]: https://developers.google.com/identity/protocols/OpenIDConnect
[oidc-discovery-url]: https://openid.net/specs/openid-connect-discovery-1_0.html#ProviderConfig
[conf-ref]: {{< ref "/docs/reference/configuration/#overriding-default-configuration-templates" >}}
