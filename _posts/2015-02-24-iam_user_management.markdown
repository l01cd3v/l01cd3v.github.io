---
layout: post
title:  "IAM user management strategy"
date:   2015-02-24 20:49
post_author: Loïc Simon
categories: AWS
tags: iSEC
---

### Use IAM groups

When granting privileges to IAM users, AWS account administrators should avoid
use of user-specific policies. Instead, create groups whose name explicitly
defines the members' job functions or responsibilities (*e.g.* AWS
Administrators, Operations, Developers, Accountants), and define the
permissions granted within group policies. Doing so will simplify the
permissions management process as changes in group policies apply to all
members.

When performing AWS configuration reviews, iSEC often discovers IAM users
whose privileges have been granted via a combination of IAM user and IAM group
policies. It is not uncommon to see IAM users who are granted full
administrator privileges in a redundant manner, via both user and group
policies. Such configuration creates an avenue for configuration mistakes, as
another administrator may believe that terminating an IAM user's membership to
the admin group is sufficient. Therefore, banning use of IAM user policies
will result in making one's AWS environment less error-prone.

***Note***: It is on purpose that iSEC recommends using IAM group names that
reflect a job title or responsibility. IAM users who do not fit in such groups
(*e.g.* headless users) should not exist. Instead, AWS account administrators
should investigate use of IAM roles for EC2. Further details will be discussed
in an upcoming blog post.

### Create a common IAM group to apply generic policies

Because a number of policies must be applied to all users, iSEC recommends that
AWS account administrators create an IAM group that all IAM users belong to.
Doing so will allow AWS account administrators to consistently grant privileges
and enforce a number of rules.

***Note***: It is important that all IAM users belong to this common IAM group
to ensure that policies are consistently applied. Failure to do so will create
gaps in one's AWS environment security posture.

### Authorize IAM users to manage their credentials

To begin with, iSEC recommends that AWS account administrators allow all of
their IAM users to manage their credentials, and only theirs. With all IAM
users belonging to the common IAM group, this can be achieved by applying the
following IAM policy to the group.

    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": [
                     "iam:*AccessKey*",
                     "iam:*Password",
                     "iam:*MFADevice*",
                     "iam:UpdateLoginProfile"
            ],
          "Resource": "arn:aws:iam::AWS_ACCOUNT_ID:user/${aws:username}"
        },
        {
          "Effect": "Allow",
          "Action": [
                     "iam:CreateVirtualMFADevice",
                     "iam:DeleteVirtualMFADevice"
          ],
          "Resource": "arn:aws:iam::AWS_ACCOUNT_ID:mfa/${aws:username}"
        }
      ]
    }

While the above policy is sufficient to allow users to manage their
credentials, AWS administrators may consider the following statement as an
addition; it allows IAM users to know what the account's password policy is.

    {
      "Effect": "Allow",
      "Action": "iam:GetAccountPasswordPolicy",
      "Resource": "*"
    }

### Use AWS Scout2 to detect user policies

The default ruleset used by [AWS
Scout2](https://github.com/iSECPartners/Scout2) includes a rule that checks for
user policies and reports the use of user-specific IAM policies as a warning.
Detection of user-specific IAM policies results in the IAM menu dropdown
containing a "User policies" security risk, as illustrated in the below screenshot.

![Screenshot: IAM menu dropdown with a User policies security risk](/images/aws/awsscout2-iam-user-policy-1.png)

When clicked-on, this "User policies" link filters the list of IAM users to only
display those who have at least one user policy attached. The orange badge
indicates a warning and the count of user policies attached to this particular
IAM user.

![Screenshot: Orange badge indicating that at least one user policy is attached to that IAM user](/images/aws/awsscout2-iam-user-policy-2.png)

### Check that all IAM users belong to the common group

AWS Scout2 comes with a tool — RulesGenerator.py — that allows AWS account
administrators to generate a custom ruleset to tailor the report to their
needs. An optional IAM rule requires all IAM users to belong to a common IAM
group. In order to enable this rule, the following can be done:

1. Run the rules generator with the following command line:
<pre style="margin-left: -20px; margin-top: 10px; margin-bottom: 10px"><code>./RulesGenerator.py --ruleset_name isec --services iam</pre></code>
1. Answer "yes" to the question "Would you like to ensure that all IAM users belong to a given IAM group?"
1. Enter the name of your common group (*e.g.* AllUsers)
1. Enter "yes" or "y" to confirm
1. Change the level if desired
1. Run Scout2

***Note***: If you have already run Scout2 and do not wish to download the latest
IAM configuration, use the following command to run an offline analysis:

    ./Scout2.py --ruleset_name isec --services iam --local

The following screenshot illustrates the IAM menu dropdown containing a
security risk when IAM users do not belong to the configured common group.

![Screenshot: IAM menu dropdown when IAM users do not belong to the common group](/images/aws/awsscout2-iam-user-commongroup-1.png)

When clicked-on, this link filters the list of IAM users to only display those
who do not belong to the common IAM group. A colored warning sign appears,
warning about this issue.

![Screenshot: Orange badge indicating that at least one user policy is attached to that IAM user](/images/aws/awsscout2-iam-user-commongroup-2.png)

### Conclusion

Strict management of IAM users and tight control of their privileges is key in
maintaining a secure AWS environment. When followed, the above recommendations
should enable AWS administrators to manage IAM users with improved efficiency
and lower the chances of overly privileged users to exist.
