---
layout: post
title:  "Use and enforce Multi-Factor Authentication"
date:   2015-04-02 14:10
post_author: Loïc Simon
categories: AWS
tags: iSEC
---

### What is Multi-Factor Authentication?

When enabled, Multi-Factor Authentication (MFA) provides strong
defense-in-depth against compromises of credentials. MFA-enabled users have a
device that periodically generates a new authentication code (*i.e.* one-time
password); they need to enter the current authentication code along with their
static credentials (*i.e.* username and password) in order to successfully
authenticate. In addition to supporting MFA when accessing the web console
(*i.e.* password-based authentication), AWS also offers MFA-protected API
access for users who work with AWS access keys. Through the Security Token
Service (STS), IAM users can request temporary credentials in exchange for
their long-lived credentials (*i.e.* AWS access key ID and secret key) and
their current authentication code.

### Why should one use and enforce MFA?

For companies deploying their application in the cloud, a breach that results
in unauthorized access to the management console &mdash; or API &mdash; is the
worst-case scenario. While a number of AWS administrators have realized the
importance of enabling MFA when they access the web console, a limited number
of them enforce MFA-protected API access. This represents a huge gap in one's
security posture because AWS access keys do not come with as many security
features as passwords do:

* AWS administrators can enforce password expiration; this is currently not
possible for AWS access keys.
* While it is probably safe to assume that most AWS administrators do not store
their password in plaintext, most of them use AWS access keys. By design, these
keys are meant to be stored in plaintext files that are accessed by tools built
with the various AWS SDKs.
* A lost password is a forgotten password; a lost key is a key stored in a lost
file, which may be on an unencrypted storage device (e.g. hard drive or USB Flash
drive).

Because AWS access keys are long-lived credentials that are stored in plaintext
files, they are more susceptible to compromise than passwords. It is therefore
necessary to enable MFA when the AWS API is accessed using these keys and not
only when users sign in using their passwords.

### How can one enforce MFA?

Unfortunately, at time of writing, AWS does not offer an option to enforce
MFA-protected API access via a global setting. Therefore, AWS account
administrators must carefully manage their IAM users and develop a strategy to
reliably achieve this. In order to enforce MFA-protected API access, iSEC
recommends the following:

1. Create a common IAM group that all IAM users belong to, as discussed in the previous [IAM user management strategy]({{ site.baseurl }}/aws/2015/02/24/iam_user_management.html) post.
1. Add the following policy (also available on [Github](https://github.com/iSECPartners/AWS-recipes/blob/master/IAM-Policies/EnforceMFA-8HourSession.json)) to enforce MFA for all users who belong to this group.


This policy will enforce MFA regardless of how the IAM user authenticated with
AWS; it will be effective whether they use password-based or key-based
authentication.

    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Deny",
          "Action": "*",
          "Resource": "*",
          "Condition": {
            "Null":{"aws:MultiFactorAuthAge":"true"}
          }
        },
        {
          "Effect": "Deny",
          "Action": "*",
          "Resource": "*",
          "Condition": {
            "NumericGreaterThan":{"aws:MultiFactorAuthAge":"28800"}
          }
        }
      ]
    }


The first statement in the above policy denies all actions if the
*aws:MultiFactorAuthAge* key is not present; this key only exists if MFA is
used [1].

The second statement verifies that the validation of the MFA code was performed
less than eight hours ago. Temporary credentials may be valid for a duration
between fifteen minutes and thirty-six hours [2]. iSEC recommends requiring
users to initiate a new session at least once a day.

***Note:*** An "explicit deny" means that, regardless of other policies granted
to a user, this deny rule will prevail. More information about the IAM policy
evaluation logic can be found in the AWS documentation at <a
target="_blank"
href="http://docs.aws.amazon.com/IAM/latest/UserGuide/AccessPolicyLanguage_EvaluationLogic.html">http://docs.aws.amazon.com/IAM/latest/UserGuide/AccessPolicyLanguage_EvaluationLogic.html</a>.

### Use AWS Scout2 to detect users without MFA

The default ruleset used by [AWS Scout2](https://github.com/iSECPartners/Scout2) includes a rule that checks for IAM users who have password-based authentication enabled but do not have an MFA device configured. If Scout2 detects IAM users with password-based authentication enabled and no MFA device, it will document a "Lack of MFA" security risk in the IAM menu dropdown, as illustrated in the below screenshot.

![Screenshot: IAM menu dropdown with a "Lack of MFA" security risk](/images/aws/awsscout2-user-nomfa-1.png)

When clicked, this "Lack of MFA" link filters the list of IAM users to display
those who have password-based authentication enabled but no MFA device
configured. The red "No" following "Multi-Factor enabled" indicates a danger
tied to that particular IAM user.

![Screenshot: Red "No" indicating that this IAM user may access the web console without MFA](/images/aws/awsscout2-user-nomfa-2.png)

### How can one use MFA with command line tools?

Users of the AWS CLI (and other command line tools) have several methods to
configure their credentials, such as environment variables, configuration
files, or command line arguments. However, updating these settings on a daily
basis when MFA-protected API access is enabled is inconvenient. To help
facilitate this work flow, iSEC has created a set of Python tools and released
them in the [AWS-recipes](https://github.com/iSECPartners/AWS-recipes)
repository. Further details about these tools will be published in the next
blog post.

Additional information about MFA with AWS is available in the AWS
documentation at <a target="_blank"
href="https://docs.aws.amazon.com/IAM/latest/UserGuide/Using_ManagingMFA.html">
https://docs.aws.amazon.com/IAM/latest/UserGuide/Using_ManagingMFA.html</a>.


### Conclusion

Enforcing Multi-Factor Authentication for all IAM users is extremely important
in order to mitigate the risks of credentials compromise (especially the AWS
access key ID and secret). This aspect of security is commonly overlooked and
may result in catastrophic damages. By using a strict strategy for management
of IAM users and the above IAM policy, AWS administrators may significantly
reduce risks of account compromise.

[1] <a target="_blank" href="http://docs.aws.amazon.com/IAM/latest/UserGuide/AccessPolicyLanguage_ElementDescriptions.html#AvailableKeys">http://docs.aws.amazon.com/IAM/latest/UserGuide/AccessPolicyLanguage_ElementDescriptions.html#AvailableKeys</a>

[2] <a target="_blank" href="http://docs.aws.amazon.com/STS/latest/APIReference/API_GetSessionToken.html">http://docs.aws.amazon.com/STS/latest/APIReference/API_GetSessionToken.html</a>
