# Deployment to AWS CloudFront using AWS Cloud Formation and Bitbucket PipeLines
It is an example of how to quickly deploy static UI code to AWS CloudFront using Bitbucket Pipe Lines.
This approach creates AWS Formation Stack or updates if one already created.
This code should be pushed to the repository on Bitbucket! `bitbucket-pipelines.yml` file will be ignored on
 other platforms!

## Configuring Bitbucket PipeLines
To make it works under Bitbucket, You need:
 - Enable Pipelines under settings section of the repository
 - Provide AWS access key and secret key of the user that is allowed to perform necessary actions under the AWS account.
 - Trigger Pipeline to do the actual deployment.

### Enable Pipelines
To enable Bitbucket Pipelines, You need to:
 - Go to the bitbucket repository;
 - Got to the settings page;
 - Got to the settings page under "PIPELINES" section;
 - Check "Enable Pipelines" checkbox

### Provide AWS user credentials
It uses AWS CLI to deploy code to AWS. So, we need to provide the credentials of the user to connect to AWS.
There are few ways to do that, but it is a bad idea to store sensitive data within the code. The better way is to store as Environment variable to Bitbucket pipelines as a secret value. It is still possible to extract this value, but it requires writing access to the repository.
To store the AWS credentials, you should:
 - Go to the bitbucket repository;
 - Go to the settings page;
 - Go to "Repository variables" section;
 - Create 3 variables:
    - `AWS_ACCESS_KEY_ID` should contain AWS access key;
    - `AWS_SECRET_ACCESS_KEY` should contain AWS secret key;
    - `AWS_DEFAULT_REGION` should contain the AWS region name.

### Trigger build
The building of the code is triggered each time changes are pushed to the repository. The first time, when the pipeline is enabled, You can trigger a build manually from UI, or You can push new changes to the repo - this triggers the build process automatically.


## First deployment

### AWS CloudFormation Deployments' data bucket
AWS CloudFormation uses the S3 bucket to upload some deployments' intermediate data. So, before triggering the first build - you need to create such a bucket for AWS CloudFormation. You should put the name of that bucket into `bitbucket-pipelines.yml` file.
That data is using only between `package` and `deploy` stages of AWS CloudFormation stack. So, best practice is to configure automatic deleting files from the bucket after some period e.g., 30 days.

### Invalidation of AWS CloudFront distribution cache
During the first deployment, there is no distribution yet. Id of the distribution is required to invalidate the AWS CloudFront cache. I commented 2 lines in `bitbucket-pipelines.yml` to avoid build issues. After the first deployment, you can use the id of created distribution and uncomment that lines. Remember that there are different distributions for different environments.
After pushing changes to the repo - this change applies automatically.
This change allows you to see changes in your UI code immediately after deployment is completed.

### Aliases configuration
Also, during the first deployment, you can not create DNS alias for your domain for the same reason. So, after the first deployment, you should create a CNAME record in the DNS hosted zone's table. Then you need to uncomment aliases section in the AWS CloudFormation template.
This change allows you to access your UI using your custom domain name instead of something like `random_string.cloudfront.net`.

### Acm certificate
Again, during the first deployment, most likely, you do not have created an ACM certificate. So, after you created that certificate, you should provide its ARN into the `bitbucket-pipelines.yml` and uncomment `Certificate` section in the AWS CloudFormation template.
This change allows you to use the HTTPS protocol.

### Disallow to use the HTTP protocol
To meet security requirements, most likely you want to disallow to use the HTTP protocol. To achieve this, you need to change `ViewerProtocolPolicy` property to `redirect-to-https` value.

## Usage
Just use it.
