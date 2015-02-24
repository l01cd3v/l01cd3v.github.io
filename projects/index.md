---
layout: not_list
---

# Projects

## Cloud Security

### AWS Scout2

Scout2 is a security tool designed to assist security assessments of AWS
environments. It is written in Python and uses boto (AWS SDK for Python) to
gather security-related information for several services such as IAM, EC2, and
S3. The tool is open-source and available on Github at
[https://github.com/iSECPartners/Scout2](https://github.com/iSECPartners/Scout2).

### AWS Recipes

The AWS Recipes repository is a placeholder for the following:

* Tools that facilitate daily work with AWS in a secure manner
* Tools that may help penetration testers who compromise an AWS access key
* IAM policies that improve one's security posture

Most tools available are written in Python, but I do not exclude using other
SDKs or the CLI tool sometimes. All tools, techniques, and policies are
discussed in the [AWS blog post series]({{ site.baseurl}}/categories/aws). The
AWS Recipes repository is available at
[https://github.com/iSECPartners/AWS-Recipes](https://github.com/iSECPartners/AWS-Recipes).

### AWS Utils

AWS Utils is a Python library shared between Scout2 and tools provided in AWS
Recipes. It is used as a git module in both repositories to allow code reused
accross projects. This library is open-source as well and is available on Github at
[https://github.com/iSECPartners/AWSUtils](https://github.com/iSECPartners/AWSUtils).

## Privacy

### CookieJar

CookieJar is a browser extension for Chrome and Opera that strips cookies out
of HTTP requests when the root domain of the target URL does not match the root
domain of the tab's URL. It has the effect of anonymizing all "silent" requests
that are sent to third parties (_e.g._ social networks) when you browse the
web. The source code is open-source and available on Github at
[https://github.com/l01cd3v/CookieJar](https://github.com/l01cd3v/CookieJar).
The released extensions are available for free in each browser's extension
store:

* Chrome : [https://chrome.google.com/webstore/detail/cookie-jar/iikhngeniebocgncjooahpaojhnonkfb](https://chrome.google.com/webstore/detail/cookie-jar/iikhngeniebocgncjooahpaojhnonkfb)
* Opera : [https://addons.opera.com/en/extensions/details/cookie-jar](https://addons.opera.com/en/extensions/details/cookie-jar)
