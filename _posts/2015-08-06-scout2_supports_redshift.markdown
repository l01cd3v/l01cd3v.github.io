---
layout: post
title:  "Redshift support added in Scout2"
date:   2015-08-06 08:39
post_author: Loïc Simon
categories: AWS
tags: NCC
---

Today, I am excited to announce that support for Redshift was added in
Scout2. By default, Scout2 will fetch information about your Redshift clusters,
cluster parameter groups, and cluster security groups if you still use
EC2-Classic. At this stage, Scout2 comes with six Redshift security rules that
are enabled by default:

   * Clusters
      * Check whether version upgrade is enabled
      * Check whether the cluster is publicly accessible
      * Check whether database encryption is enabled
   * Cluster parameter groups
      * Check whether SSL/TLS is required to access the database
      * Check whether user activity logging is enabled
   * Cluster security groups (EC2-classic)
      * Check whether the security group allows access to all IP addresses (0.0.0.0/0)

Scout2 was first released over a year and a half ago, and proved to be extremely
helpful when performing AWS configuration reviews. While
Scout2's initial release only supported three services (IAM, EC2, and S3) and
included thirteen security checks, the tool rapidly grew to add support for RDS
and CloudTrail. Furthermore, the tool now offers over fifty tests throughout
these five supported services. I hope that support for Redshift will bring
value to users of Scout2, and welcome feature requests, bug reports, and
recommendations on Github at
[https://github.com/iSECPartners/Scout2/issues](https://github.com/iSECPartners/Scout2/issues).
