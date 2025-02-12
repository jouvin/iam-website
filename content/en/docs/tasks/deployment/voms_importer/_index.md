---
title: "Importing VOMS information into IAM"
weight: 140
---

## The VOMS importer script

For VOs that already use VOMS to manage their organization, a script is
available to synchronize one-way their VOMS instance with their IAM instance.

The VOMS information that is synchronized includes:

* VOMS Groups
* VOMS Roles
* VOMS Users
  * Personal information
  * X.509 certificates
  * Group and role membership
  * Generic attributes

The script needs admininistrative privileges on both VOMS and IAM, represented
respectively by a VOMS proxy and an IAM access token. As it synchronizes INDIGO
IAM information with VOMS information (rather than overwriting it), it can be run
multiple times to fix problems in previous runs.

The documentation on how to run the `vomsimporter.py` script is available at
https://github.com/indigo-iam/voms-importer.

## Migration of groups and roles

Both IAM and VOMS support the concepts of _group_ and _role_.

Groups are very similar in both contexts. They differ slightly in the
representation: a VOMS group has the form `/<group>/<subgroup>/...`, whereas a
IAM group has the form `<group>/<subgroup>/...`, i.e. there is no leading `/`.

For example, the VOMS group `/atlas/production` becomes the IAM group
`atlas/production`.

Roles differ in a more profound way: whereas a VOMS role is a distinct entity
from a group, in IAM a role is simply a group with the specific labels
`wlcg.optional-group` and `voms.role`. An optional group is included in an OAuth
token or in a VOMS proxy only if requested.

For example, the VOMS group with role `/atlas/Role=VO-Admin` becomes the IAM
optional group `atlas/VO-Admin`.

Groups and roles are imported in IAM only if not already present there.

Groups and roles can be created directly in IAM without any corresponding group
or role in VOMS; those are not considered by the importer script.

## Migration of users

Only **active** VOMS users are imported: nothing is done on the INDIGO IAM side
for a user suspended in VOMS, except if you add option ` --synchronize-activation-status`
to force the synchronization of activation state in INDIGO IAM using the activation
state in VOMS. **Without this option, a user suspended in VOMS may still be
active in IAM if the user was imported (and active) before it was suspended in VOMS.** 

*Note about CERN instances: since INDIGO IAM at CERN is also integrated with the CERN
Human Resource (HR) database, the HR check will ensure that users are
suspended/removed from the VO when experiment membership is no longer valid.*

When creating a IAM account for a newly-discovered VOMS user, the importer
script generates a username of the form `voms.<voms-id>`. There is an option to
use the value of an attribute (e.g. `nickname`) from the user entry in VOMS, if
available.

The script by default removes a user from groups (including roles) where they do
not belong any more.

### CERN SSO automatic linking

The importer script can be configured to automatically link the imported account
to the CERN SSO identity in two ways:

1. using the `nickname` VOMS account generic attribute
2. resolving the CERN username from the CERN LDAP

### Unique email address for users in IAM

VOMS allows different users to share an email address. IAM instead requires that
each user has a unique email address.

The import script implements an email override mechanism, that maps a VOMS user
id to another email address; however, a better approach would be to clean up the
VOMS database in order to avoid multiple accounts sharing email addresses.

## VOMS importer log

The importer script produces an auditing log, detailing the actions applied to
groups, roles and users.
