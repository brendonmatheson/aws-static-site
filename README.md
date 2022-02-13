# AWS Static Site

This CloudFormation template creates static site hosting in AWS S3 with logging and redirectors and with TLS managed in Certificate Manager and edge caching with CloudFront.

For more information see:

- Original blog post at https://brendonmatheson.com/2019/05/23/replatforming-to-jekyll-and-aws-s3.html
- Follow-up blog post at https://brendonmatheson.com/2020/08/02/static-site-with-s3-cloudfront.html

This code is offered under the terms of the GNU General Public License v2.  See LICENSE for details.

## Default Document Mapping

### Introduction

This project provides the code for mapping default documents for paths served through a static website.

CloudFront with a S3 REST origin does not support default docuemnt as discussed in this [older solution based on Lambda@Edge](https://aws.amazon.com/blogs/compute/implementing-default-directory-indexes-in-amazon-s3-backed-amazon-cloudfront-origins-using-lambdaedge/):

> However ... CloudFront must use the S3 REST endpoint to fetch content from your origin so that the request can be authenticated using the OAI. This presents some challenges in that the REST endpoint does not support redirection to a default index page.
>
> CloudFront does allow you to specify a default root object (index.html), but it only works on the root of the website (such as http://www.example.com > http://www.example.com/index.html). It does not work on any subdirectory (such as http://www.example.com/about/). If you were to attempt to request this URL through CloudFront, CloudFront would do a S3 GetObject API call against a key that does not exist.

### CloudFront Functions

From the [CloudFront Functions announcement blog post](https://aws.amazon.com/blogs/aws/introducing-cloudfront-functions-run-your-code-at-the-edge-with-low-latency-at-any-scale/):

> CloudFront Functions are ideal for lightweight processing of web requests, for example:
>
> - Cache-key manipulations and normalization: Transform HTTP request attributes (such as URL, headers, cookies, and query strings) to construct the cache-key, which is the unique identifier for objects in cache and is used to determine whether an object is already cached. For example, you could cache based on a header that contains the end user’s device type, creating two different versions of the content for mobile and desktop users. By transforming the request attributes, you can also normalize multiple requests to a single cache-key entry and significantly improve your cache-hit ratio.
> - URL rewrites and redirects: Generate a response to redirect requests to a different URL. For example, redirect a non-authenticated user from a restricted page to a login form. URL rewrites can also be used for A/B testing.
> - HTTP header manipulation: View, add, modify, or delete any of the request/response headers. For instance, add HTTP Strict Transport Security (HSTS) headers to your response, or copy the client IP address into a new HTTP header so that it is forwarded to the origin with the request.
> - Access authorization: Implement access control and authorization for the content delivered through CloudFront by creating and validating user-generated tokens, such as HMAC tokens or JSON web tokens (JWT), to allow/deny requests.

References:

- [Introducing CloudFront Functions – Run Your Code at the Edge with Low Latency at Any Scale](https://aws.amazon.com/blogs/aws/introducing-cloudfront-functions-run-your-code-at-the-edge-with-low-latency-at-any-scale/)
- [Add index.html to request URLs that don’t include a file name](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/example-function-add-index.html)

### Rewrite Function

Rewriting URL's from directory paths to default documents is done by a CloudFront Function that consumes the Viewer Request event from CloudFront.

The function is provided in the CloudFormation template [default-document-function.yaml](default-document-function.yaml).  This template creates the function and publishes the ARN of the created function version in the CloudFormation export `default-document-function-arn`.

Deploy the function using either the CloudFormation console or CLI and give it the stack name `default-document-function-arn`.

References:

- [Add index.html to request URLs that don’t include a file name](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/example-function-add-index.html)
