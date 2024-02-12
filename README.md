# Implementing Prerender on Existing AWS CloudFront Distributions: A Step-by-Step Guide

## Why Implement Prerender?

In the quest to improve our site's SEO and performance, the idea of implementing prerendering on our AWS CloudFront distribution came up. Prerendering is pivotal for serving pre-rendered pages to search engine crawlers, ensuring our dynamic content is indexed correctly. However, we faced a challenge: our CloudFront distributions were already set up with default behaviors for handling our content, primarily served from an S3 bucket. Existing documentation, like the one from a community member's [GitHub post](https://github.com/jinty/prerender-cloudfront), offered solutions that required starting from scratch, which wasn't feasible for our setup. Neither CloudFormation nor Terraform provided a straightforward way to modify existing distributions directly. This prompted a journey of experimentation to find a solution.

## How We Did It

After some trial and error, we devised a method using AWS CloudFormation to create and manage the necessary Lambda functions and roles, integrating them with our existing CloudFront distribution. This approach allowed us to maintain our current setup while incorporating prerendering capabilities.

### Prerequisites

Before diving into the instructions, ensure you have an existing CloudFront distribution pointing to your S3 bucket as the origin. It's crucial that your files or the entire bucket are publicly readable.

### Step-by-Step Instructions

#### 1. Deploying the CloudFormation Stack

- **Upload the YAML File**: Start by uploading the provided YAML file to AWS CloudFormation. This file contains the necessary resources for the prerendering setup. You'll be prompted to enter the token you received from prerender.io.
- **Wait for Completion**: After uploading the file and entering your details, wait for the CloudFormation stack to finish its deployment.

#### 2. Configuring Lambda Functions

- **Accessing Lambda Functions**: Go to the CloudFormation's Resources tab upon completion. You'll find links to `RedirectToPrerender` and `SetPrerenderHeader` Lambda functions. Open these links and navigate to the Versions tab in the Lambda interface.
- **Note the ARN**: Click on the version number "1" for each function to view their ARNs (Amazon Resource Names). Keep these ARNs handy as you'll need them for configuring CloudFront behaviors.

#### 3. Modifying CloudFront Distribution

- **Create a New Behavior**: In the AWS Management Console, navigate to your CloudFront distribution, and go to the Behaviors tab. Click on "Create Behavior" and set it up as follows:
    - **Path Pattern**: `*` to apply the behavior to all requests.
    - **Origin**: Select your S3 static website origin.
    - **Compress Objects Automatically**: Yes.
    - **Viewer Protocol Policy**: HTTP and HTTPS.
    - **Allowed HTTP Methods**: GET, HEAD.
    - **Restrict Viewer Access**: No.
    - **Cache Key and Origin Requests**: Choose legacy cache settings and include the following custom headers: `X-Prerender-Cachebuster`, `X-Prerender-Token`, `X-Prerender-Host`, and `X-Query-String`.
    - **Object Caching**: Customize with Minimum TTL, Maximum TTL, and Default TTL all set to 31536000.
    - **Function Associations**:
        - For Viewer Request and Origin Request, select Lambda@Edge and input the ARNs for `SetPrerenderHeader` and `RedirectToPrerender` respectively, appending `:1` to signify version 1.

#### 4. Finalizing and Testing

Once you've set up the CloudFront behaviors and Lambda function associations, test your prerender implementation to ensure everything is working correctly:

- **Test the Prerendered Page**: Use `curl` with a Googlebot user agent to simulate a search engine crawler accessing your page. Replace `${CLOUDFRONT_DOMAIN}` with your CloudFront domain or Route 53 route:

    `curl -H 'User-Agent: googlebot' https://${CLOUDFRONT_DOMAIN}/over/here`
    
    This should display the prerendered version of the page.

- **Compare with the Regular Page**: To see the difference prerendering makes, compare the above output with a standard request:

    `curl https://${CLOUDFRONT_DOMAIN}/over/here`
    
    This shows how the page appears to regular users, without prerendering.

By comparing these two responses, you can confirm the effectiveness of your prerender setup in delivering SEO-friendly, prerendered content to crawlers while serving the original content to regular users.

## Conclusion

Integrating prerendering into an existing CloudFront distribution doesn't have to involve starting from zero. With some CloudFormation magic and manual tweaking, we've managed to enhance our SEO without overhauling our current infrastructure. This guide should help you achieve the same, ensuring your dynamic content is as visible as possible to search engines.
