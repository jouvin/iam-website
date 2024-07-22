---
title: "Handling SCIM response"
linkTitle: "SCIM response"
weight: 6
---

IAM already supports a set of [SCIM APIs](../../api/scim-api/_index.md) based on the System for Cross-domain
Identity Management standard. Starting from IAM v1.9.0, the API response to GET requests on user details
(i.e. `/scim/Me` and `/scim/Users`) are extended with more information.
All the new details are included into the _IndigoUser_ field.
To enable this feature, dedicated properties (or environment variables) must be used to maintain backward compatibility.

The list of information that may be included into the SCIM response are:

* user's autorities
* set of user-managed groups
* set of user's attributes.


### Authorities

In order to include the list of user's authorities (i.e. `ROLE_USER`, `ROLE_ADMIN`, `ROLE_GM:<group-uid>`, etc.) into the SCIM response,
one should set the following property (default is false):

```yaml
scim:
  include_authorities: true
```

or the environment variable `IAM_SCIM_INCLUDE_AUTHORITIES=true`.

Then, an example of SCIM Me output for an administrator user is the following:

```
{
  "id": "73f16d93-2441-4a50-88ff-85360d78c6b5",
  "meta": {  ...  },
  "schemas": [
      "urn:ietf:params:scim:schemas:core:2.0:User",
      "urn:indigo-dc:scim:schemas:IndigoUser"
  ],
  ...
  "userName": "admin",
  ...
  "active": true,
  "urn:indigo-dc:scim:schemas:IndigoUser": {
    "oidcIds": [ ... ],
    "samlIds": [ ... ],
    "certificates": [ ... ],
    "authorities": [
      "ROLE_ADMIN",
      "ROLE_USER"
    ]
  }
}
```


### Managed groups

In order to include the list of user-managed groups into the SCIM response,
one should set the following property (default is false):

```yaml
scim:
  include_managed_groups: true
```

or the environment variable `IAM_SCIM_INCLUDE_MANAGED_GROUPS=true`.

Then, an example of SCIM Me output for a group manager is the following:

```
{
  "id":"80e5fb8d-b7c8-451a-89ba-346ae278a66f",
  "meta":{ ... },
  "schemas":[
     "urn:ietf:params:scim:schemas:core:2.0:User",
     "urn:indigo-dc:scim:schemas:IndigoUser"
  ],
  ...
  "userName": "gm-user",
  ...
  "active": "true",
  "urn":"indigo-dc:scim:schemas:IndigoUser":{
     "oidcIds":[ ... ]
     "samlIds":[ ... ],
     "endTime": "2024-08-06T02:00:00.000+02:00",
     "managedGroups":[
        {
           "display":"Analysis",
           "value":"6a384bcd-d4b3-4b7f-a2fe-7d897ada0dd1",
           "$ref":"http://localhost:8080/scim/Groups/6a384bcd-d4b3-4b7f-a2fe-7d897ada0dd1"
        },
        {
           "display":"Production",
           "value":"c617d586-54e6-411d-8e38-64967798fa8a",
           "$ref":"http://localhost:8080/scim/Groups/c617d586-54e6-411d-8e38-64967798fa8a"
        }
     ]
  }
}
```

### Attributes

In order to include the list of user's attributes, selected by attribute name, into the SCIM response,
one should set the following property (default is false):

```yaml
scim:
  include_attributes:
  - name: nickname
  - name: another-attribute-name
```

Then, an example of SCIM Me output for an IAM user is the following:

```
{
  "id":"d836e5ec-246c-456c-8476-923ee2f831c8",
  "meta":{ ... },
  "schemas":[
     "urn:ietf:params:scim:schemas:core:2.0:User",
     "urn:indigo-dc:scim:schemas:IndigoUser"
  ],
  ...
  "userName": "tester",
  ...
  "active": "true",
  "urn":"indigo-dc:scim:schemas:IndigoUser":{
    "oidcIds":[ ... ]
    "samlIds":[ ... ],
    "endTime": "2024-08-06T02:00:00.000+02:00",
    "attributes":[
       {
          "name":"nickname",
          "value":"tester"
       }
    ],
    "endTime": "2024-08-06T02:00:00.000+02:00"
  }
}
```

In this example the user has only one attribute, named _nickname_, so the second attribute (_another-attribute-name_) is not shown into the SCIM response.
