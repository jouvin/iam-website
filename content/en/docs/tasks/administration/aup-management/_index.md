---
title: Acceptable Usage Policy (AUP) management
weight: 3
---

Starting with version 1.2.0, IAM supports defining an **Acceptable Usage Policy
Document (AUP)** for a managed organization. The AUP is presented to applicants
at registration time or periodically. 

## Managing the AUP 

The AUP can be managed from the "AUP" section in the IAM dashboard, or via the
[AUP management API]({{< ref "/docs/reference/api/aup-api" >}}). 

Administrator privileges are required.

### Creating the AUP

By default, the AUP is not defined for an organization, i.e. it must be
explicitly defined by an IAM administrator. 

The AUP can be created by clicking on the "Create AUP" button in the AUP
management page (that can be reached clicking the "AUP" link in the organization
Management navigation bar):

![AUP management 0](../images/aup0.png)

A dialog is shown:

![AUP management 1](../images/aup1.png)

where the administrator can enter the AUP URL and define how frequently the
AUP should be accepted by users.

If the AUP signature validity is set to 0, a signature on the AUP will be
requested only once (either at registration time or at the first IAM login
after the AUP creation). A zero value means basically that the AUP signature
does not expire.

A positive value in the AUP signature validity means, on the other hand, that
the signature will expire after that number of days. As an example, an
administrator that wants the AUP to signed each year by users should put 365 in
the "AUP signature validity" field.

### Editing and deleting the AUP

The AUP can be edited and deleted from the AUP management page:

![AUP management 2](../images/aup2.png)

Since version 1.6.0, the AUP has been converted from text to a URL.
If you have just upgraded to 1.6.0 and you have already defined a text AUP,
you should see the following red message below your AUP:

![AUP management 1b](../images/aup1b.png)

By clicking on "Edit AUP", admins are prompted to insert a valid URL.

![AUP management 5](../images/aup5.png)

As shown in the picture above, editing the AUP will not trigger an AUP signature request.

### Request a signature from users

If you want to request a signature from users for the updated AUP, 
use the "Request AUP signature" button in the AUP management page.

A dialog is shown:

![AUP management 6](../images/aup6.png)

Confirm by clicking on "Request AUP signature" and the acceptance of the AUP
will be requested from ALL users at their next login.

## Signing the AUP

Once defined the AUP text is shown to users for acceptance either
at registration time:

<img src="../images/aup3.png" width="500">

or at the first login after the AUP creation:

<img src="../images/aup4.png" width="500">

## Request AUP signature

Administrator can request user to sign AUP.

Administrator privileges are required.

Log into the service using admin credentials and click on the _Users_ link on the left navigation bar:

![dashboard](../images/request-aup-signature-1.png)

From the _Users_ link, select _Any user_ you want to request AUP signature, for example _Test-200 User_:

![user](../images/request-aup-signature-2.png)

To request AUP signature click on _Request AUP signature_ button on _User details_ section on the left of the page:

![request AUP signature button](../images/request-aup-signature-3.png)

To confirm your choice click on _Request AUP signature_ button on the modal window:

![request AUP signature modal](../images/request-aup-signature-4.png)

On success you will get a confirmation message:

![request AUP signature confirmation](../images/request-aup-signature-5.png)

After this operation, if the user logs in, they must sign the AUP again before accessing the service.

## Sign AUP on behalf of the user

Administrator can sign AUP on behalf of the user.

Log into the service using admin credentials and click on the _Users_ link on the left navigation bar:

![dashboard](../images/request-aup-signature-1.png)

From the _Users_ link, select _Any user_ you want to sign AUP on behalf, for example _Test-200 User_:

![user](../images/request-aup-signature-2.png)

To sign AUP on behalf of the user click on _Sign AUP on behalf of this user_ button on _User details_ section on the left of the page:

![sign AUP on behalf button](../images/sign-aup-on-behalf-1.png)

To confirm your choice click on _Sign AUP_ button on the modal window:

![sign AUP on behalf modal](../images/sign-aup-on-behalf-2.png)

On success you will get a confirmation message:

![sign AUP on behalf confirmation](../images/sign-aup-on-behalf-3.png)