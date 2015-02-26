---
layout: isec
title:  "Do not use your AWS root account"
date:   2015-02-23 08:42
post_author: Loïc Simon
categories: AWS
---

### What is the AWS root account?

The AWS root account is the account that was used -- or created -- when signing
up with Amazon Web Services. This account has full access to all resources in
the account and it is not possible to alter this configuration.

### Risks of using the AWS root account

Using the AWS root account means that there is potential for its compromise.
In particular, iSEC noticed that AWS customers who use the AWS root account
tend to do the following:

1. Share credentials between employees.
1. Disable Multi-Factor Authentication (MFA) for convenience.

Shared credentials, aside from increasing the risk of compromise during the
sharing process, render credential rotation impractical due to the need for the
newly-generated secret to be known by multiple parties. Sharing the AWS root
account also undermines any effort towards using IAM and leveraging the
fine-grained access controls it offers. Finally, shared credentials result in
loss of the attribution ability, which makes auditing harder and may prevent
successful investigation.

### AWS Identity and Access Management (IAM)

AWS IAM allows account administrators to create users for every employee and
grant them access to a limited set of services, actions, and resources. This
allows AWS account administrators to apply the principle of least privilege,
which dictates that a given user should only be able to access the information
and resources that are necessary for them to perform tasks they are responsible
for. Additionally, use of IAM allows AWS users to rotate credentials and revoke
privileges without impacting other employees.

AWS account administrators should create an *Administrator* IAM group, grant
administrator privileges to this group, and create individual IAM users for
each employee in charge of administrating the AWS account. When done, the AWS
root password should be rotated and stored in a safe manner. Furthermore,
additional credentials such as access keys and certificates should be deleted.

### Important security consideration about the root account

AWS users should always enable MFA on their root account, even when the
password is securely stored; it is important to realize that the password reset
for the root account process only requires access to the email address
associated with this account. **This means that, without MFA, your production
environment is only as secure as an email.**
