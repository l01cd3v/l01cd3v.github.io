---
layout: post
title:  "Introducing opinel: Scout2's favorite tool"
date:   2015-08-03 11:08
post_author: Loïc Simon
categories: AWS
tags: iSEC
---

With boto3 being stable and generally available<sup>[1]</sup>, I took the opportunity
to migrate Scout2 and AWS-recipes to boto3. As part of that migration
effort, I decided to publish the formerly-known-as AWSUtils repository -- used
by Scout2 and AWS-recipes -- as a python package required by these tools,
rather than requiring users to work with Git submodules. I've also added more
flexibility when working with MFA-protected API calls and improved versioning
across the project.

### opinel

To avoid name conflicts, I decided to rename the shared AWSUtils code to a
less misleading name: opinel. The opinel package is published on [PyPI](https://pypi.python.org/pypi/opinel), and thus can
be installed using pip and easy\_install. The corresponding source code is still
open-sourced on Github at [https://github.com/iSECPartners/opinel](https://github.com/iSECPartners/opinel).
As a result, Scout2 and AWS-recipes have been modified to list opinel as a
requirement, which significantly simplifies installation and management of this
shared code.

### Support for Python 2.7 and 3.x

Because boto3 supports both Python2 and Python3, I decided to make sure that
the code built on top of that package has similar properties. As a result,
the latest versions of Scout2 and AWS-recipes support Python 2.7 and 3.x.
Note that opinel will **NOT** work with Python 2.6.

### Modification of the MFA workflow

As requested by a user of AWS-recipes<sup>[2]</sup>, I modified the workflow when
using MFA-protected API access to no longer store the long-lived credentials
in a separate file. As a result, the *.aws/credentials.no-mfa* file is no
longer supported and all credentials are stored in the standard AWS credentials
file under *.aws/credentials*. Usage of the existing tools remains unchanged,
but the long-lived credentials are now accessible via a new profile name:
*profile\_name-nomfa*. This allows users to work with both STS and long-lived
credentials if need be.

If you already had configured your environment to work with MFA-protected API
access, you will need to copy your long-lived credentials back to the
*.aws/credentials* file. This can be done with a simple command such as the
following:

    cat ~/.aws/credentials.no-mfa | sed -e 's/]$/-nomfa]/g' >> ~/.aws/credentials

### Support to use assumed-role credentials

With this new workflow implemented, I created a new recipe that allows
configuration of role-credentials in the *.aws/credentials* file. When the following
command is run, it uses the credentials associated with the *isecpartners*
profile to request role credentials for the IAM-Scout2 role. The role
credentials are then written in the *.aws/credentials* file in a new profile
named *isecpartners-Scout2*, which is the profile name appended by the role
session name.

    $ ./aws_recipes_assume_role.py --profile isecpartners --role-arn arn:aws:iam::AWS_ACCOUNT_ID:role/IAM-Scout2 --role-session-name Scout2

Users can then use their favorite tools that support profiles. For example,
Scout2 could be run with the following command line:

    $ ./Scout2.py --profile isecpartners-Scout2

Note that this recipe supports MFA if the assumed role requires it:

   * If you never configured your environment to work with MFA, you can provide your MFA serial number (ARN) and current token code as arguments.
   * If you already configured your environment to work with MFA and stored your MFA serial in the *.aws/credentials* file, you just need to pass your token code as an additional argument.
   * Finally, if you already initiated an STS session, you do not need to provide a new token code and can run the command as above.

### Conclusion

With the release of opinel, I hope to simplify distribution and management of
the code shared between Scout2 and AWS-recipes. Additionally, I
significantly modified the workflow and credentials storage when working with
MFA-protected API calls, which allows users to use both their long-lived and STS
credentials.

[1]: https://aws.amazon.com/about-aws/whats-new/2015/06/boto3-aws-sdk-for-python-version-3-is-now-generally-available

[2]: https://github.com/iSECPartners/opinel/issues/4
