---
layout: post
title:  "IAM user management strategy (part 2)"
date:   2015-06-09 09:20
post_author: Loïc Simon
categories: AWS
tags: iSEC
---

The previous [IAM user management strategy]
({{{site.baseurl}}/aws/2015/02/24/iam_user_management.html) post discussed how
usage of IAM groups enables AWS administrators to consistently grant privileges
and enforce a number of security rules (such as MFA-protected API access). This
blog post will build on this idea by introducing category groups and
documenting new tools to improve IAM user management.

### Categorize your IAM users

For a variety of reasons, applying a single set of security rules to all IAM
users is not always practical. For example, because many applications running
in AWS predate IAM roles, numerous environments still rely on the existence of
headless IAM users. Additionally, third parties may be granted access to an AWS
account for a number of reasons but may not be able to comply with the same set
of security rules that employees follow. For this reason, NCC recommends using
category groups to sort IAM users and reliably enforce appropriate security
measures. For example, one group for all human users and a second for all headless users may be
created: MFA-protected API access and password management are not relevant for
headless users. Furthermore, human users may be categorized into several groups
such as employees and contractors: API access can be restricted to the
corporate IP range for employees but might not be achievable for contractors.

*Note 1:* The set of category groups should define all types of IAM users that
may exist in your AWS account and each IAM user should belong to one -- and
only one -- category group (they may belong to other groups though).

*Note 2:* The common group and category groups should be used to enable enforcing
security in one's AWS environment. Policies attached to these groups should be
carefully reviewed and grant the minimum set of privileges necessary for this
type of IAM user (*e.g.* credential management for humans).

### Example of category groups

The rest of this article describes a number of tools developed and used by
NCC to help implement this IAM user management strategy. These tools can be found
in the [AWS-Recipes](https://github.com/iSECPartners/AWS-recipes) repository. We will 
use our test AWS environment as an example, in which we use three category groups in
addition to the *AllUsers* common group:

1. *AllHumans*, the group all employees must belong to.
1. *AllHeadlessUsers*, the group all headless IAM users must belong to.
1. *AllMisconfiguredUsers*, a placeholder for sample misconfigured users.

We also have an IAM user naming convention that requires usernames to match the
following schema:

1. Employees: firstname initial appended with lastname
1. Headless user: name of the service prefixed with *HeadlessUser-*
1. Misconfigured: description of the misconfiguration prefixed with *MisconfiguredUser-*

Based on these rules, we created a configuration file stored under
*.aws/recipes/isecpartners.json*, with *isecpartners* matching the profile's
name. If you do not use profiles, the configuration will be under
*.aws/recipes/default.json*.

    {
        "common_groups": [ "AllUsers" ],
        "category_groups": [
            "AllHumanUsers",
            "AllHeadlessUsers",
            "AllMisconfiguredUsers"
        ],
        "category_regex": [
            "",
            "^Headless-(.*)",
            "^MisconfiguredUser-(.*)"
        ],
        "profile_name": [ "isecpartners" ]
    }


This configuration file declares the name of the common IAM group and two lists
related to the categorization of IAM users:

1. A list of category groups.
1. A list of regular expressions matching our naming convention.


*Note 1:* If you do not have a naming convention in place to distinguish the
type of user, remove the *category_regex* attribute from your configuration
file.

*Note 2:* If a regular expression is only applicable to a subset of category
groups, you must ensure that both lists have the same length and use an empty
string for groups that cannot be automatically associated (see the
*AllHumanUsers* group in our example).

*Note 3:* Use of a configuration file is not necessary as all values may be
passed as command line arguments. If a configuration file exists and a value is
passed as an argument, the value passed via the command line will be used.

### Create your default groups with *[aws\_iam\_create\_default\_groups.py](https://github.com/iSECPartners/AWS-recipes/blob/master/Python/aws_iam_create_default_groups.py)*

The purpose of this tool is to create IAM groups whose name matches the common
and category groups specified in the above configuration file. Running the
following command results in four new groups being created if they did not
already exist.

    ./aws_iam_create_default_groups.py --profile isecpartners

### (Automatically) sort IAM users with *[aws\_iam\_sort\_users.py](https://github.com/iSECPartners/AWS-recipes/blob/master/Python/aws_iam_sort_users.py).*

This tool iterates through all IAM users and attempts to automatically detect
the IAM groups each user should belong to. For convenience, we recommend adding
the following to your AWS recipes configuration files:

    "aws_sort_users.py": {
        "create_groups": false,
    },
    "force_common_group": true

This specifies default values for additional arguments to be set when running
*aws\_iam\_sort\_users.py*. Specifically, with these values, running this tool
will automatically add all IAM users to the common group *AllUsers* and will
not attempt to create the default groups (not necessary as we already did
this). Additionally, this tool checks that each IAM user belongs to one of the
category groups. If this is not the case and the username matches a regular
expression, the user is automatically added to the matching category group. Otherwise, a
multi-choice prompt appears to allow manual selection of the appropriate
category group.

### Additional advantages of configuration files

Besides helping with simplification of these tools' usage, this new AWS-recipe
configuration file can be used across tools, allowing for more consistent
rule enforcement. For example, the
*[aws\_iam\_create\_user.py](https://github.com/iSECPartners/AWS-recipes/blob/master/Python/aws_iam_create_user.py).*
tool uses this configuration file and applies the same business logic to add
users to the common group and appropriate category group at user creation time. In
our test environment, for example, running the following command automatically
added the new user to the *MisconfiguredUser* group:

    $ ./aws_iam_create_user.py --profile isecpartners --users MisconfiguredUser-BlogPostExample
    Creating user MisconfiguredUser-BlogPostExample...
    Save unencrypted value (y/n)? y
    User 'MisconfiguredUser-BlogPostExample' does not belong to the mandatory common group 'AllUsers'. Do you want to remediate this now (y/n)? y
    User 'MisconfiguredUser-BlogPostExample' does not belong to any of the category group (AllHumanUsers, AllHeadlessUsers, AllMisconfiguredUsers). Automatically adding...
    Enabling MFA for user MisconfiguredUser-BlogPostExample...

### Conclusion

While efficient and reliable management of IAM users can be challenging, using
the right strategy and tools significantly simplifies this process. Creation
and use of a naming convention for IAM users enables
automated user management and enforcement of security rules.

