---
layout: post
title:  "Work daily with enforced MFA-protected API access"
date:   2015-04-03 14:10
post_author: Loïc Simon
categories: AWS
tags: iSEC
---

### AWS Security Token Service 

The AWS Security Token Service (STS) is the gateway used to create sessions
when MFA-protected API access is enabled. This service allows IAM users to
retrieve short-lived credentials (*i.e* access key ID, secret access key, and
session token) in exchange for their long-lived credentials (*i.e.* AWS access
key ID and secret key) and their current authentication code. When enforcing
MFA-protected API access, as recommended in the previous [Use and enforce
Multi-Factor
Authentication]({{site.baseurl}}/aws/2015/04/02/use_and_enforce_mfa.html) post,
IAM users must use these short-lived credentials to access other AWS services.

### Challenges with MFA-protected API access

When MFA-protected API access is enforced, managing AWS access keys becomes
challenging because configuration files that contain these credentials must be
updated regularly. Users must also ensure that they do not lose their
long-lived credentials when modifying the configuration files to write their
short-lived credentials. In order to help with this workflow, iSEC wrote and
released several simple tools in the
[AWS-recipes](https://github.com/iSECPartners/AWS-recipes) repository.

The collection of tools that we will discussed below uses the "<a
target="_blank"
href="https://blogs.aws.amazon.com/security/post/Tx3D6U6WSFGOK2H/A-New-and-Standardized-Way-to-Manage-Credentials-in-the-AWS-SDKs">new
and standardized way to manage credentials in the AWS SDKs</a>", meaning that
SDKs are expecting to read credentials from the *.aws/credentials* file under
the user's home or 	profile directory.

### aws\_recipes\_configure\_iam.py

The
[aws\_recipes\_configure\_iam.py](https://github.com/iSECPartners/AWS-recipes/blob/master/Python/aws_recipes_configure_iam.py)
tool allows users to configure and store their long-lived credentials in a new,
non-standard, *.aws/credentials.no-mfa* file. In addition to prompting for the
AWS access key ID and secret key, this tool also prompts for the MFA device
serial number because this information must be provided when making calls to
the STS API. Similar to the AWS CLI and SDKs, it supports profile names. The
following code snippet is an example of calling this tool to configure a new
profile called *isecpartners*:

    $ ./aws_recipes_configure_iam.py --profile isecpartners
    AWS Access Key ID: AWS_KEY_ID
    AWS Secret Access Key: AWS_SECRET_KEY
    AWS MFA serial: arn:aws:iam::AWS_ACCOUNT_ID:mfa/USER_NAME

When looking at the *.aws* folder, we can see that a *credentials.no-mfa* file
exists and that it contains the credentials that were just entered:

    $ ls -l ~/.aws
    total 4
    -rw-r--r-- 1 loic loic 93 Apr  3 14:00 credentials.no-mfa
    $ cat ~/.aws/credentials.no-mfa
    [isecpartners]
    aws_access_key_id = AWS_KEY_ID
    aws_secret_access_key = AWS_SECRET_KEY
    aws_mfa_serial = arn:aws:iam::AWS_ACCOUNT_ID:mfa/USER_NAME

Now that long-lived credentials are configured, we can use the next tool to
call the AWS STS API and request short-lived credentials that will be used
to access other AWS services.

###  aws\_recipes\_init\_sts\_session.py

The
[aws\_recipes\_init\_sts\_session.py](https://github.com/iSECPartners/AWS-recipes/blob/master/Python/aws_recipes_init_sts_session.py)
tool reads long-lived credentials configured in the .aws/credentials.no-mfa
file, prompts users for their MFA code, and retrieves STS credentials (AWS
access key ID, AWS secret key, and session token). The short-lived credentials
are then saved under the standardized *.aws/credentials* file to be accessible
to the AWS CLI and other tools built with the AWS SDKs. The following code
snippet demonstrates calling this tool to request an STS session token:

    $ ./aws_recipes_init_sts_session.py --profile isecpartners
    Enter your MFA code: 123456
    Successfully configured the session token for profile 'isecpartners'.

When looking at the *.aws* folder, we can see that a standard *credentials*
file now exists as well and that it contains the short-lived credentials:

    $ ls -l ~/.aws
    total 8
    -rw-r--r-- 1 loic loic 576 Apr  3 14:14 credentials
    -rw-r--r-- 1 loic loic 179 Apr  3 14:00 credentials.no-mfa
    $ cat ~/.aws/credentials
    [isecpartners]
    aws_access_key_id = STS_KEY_ID
    aws_secret_access_key = STS_SECRET_KEY
    aws_mfa_serial = arn:aws:iam::AWS_ACCOUNT_ID:mfa/USER_NAME
    aws_session_token = AWS//////////SESSION_TOKEN

Now that the short-lived credentials are configured, we can use the AWS CLI or
other tools built with the AWS SDKs that read credentials from this standard
location. When the STS session expires, users just need to re-run the
[aws\_recipes\_init\_sts\_session.py](https://github.com/iSECPartners/AWS-recipes/blob/master/Python/aws_recipes_init_sts_session.py)
tool and the standard *credentials* file will be updated with new valid
short-lived credentials.

### Conclusion

By using this pair of tools to manage their AWS access keys, IAM users can
easily use the AWS CLI and other tools built with various AWS SDKs in
environments that have been secured and enforce MFA-protected API access.
