<!--
title: Serverless Framework - AWS Lambda Guide - Functions
menuText: Functions
menuOrder: 5
description: How to configure AWS Lambda functions in the Serverless Framework
layout: Doc
-->

<!-- DOCS-SITE-LINK:START automatically generated  -->
### [Read this on the main serverless docs site](https://www.serverless.com/framework/docs/providers/aws/guide/functions)
<!-- DOCS-SITE-LINK:END -->

# AWS - Functions

If you are using AWS as a provider, all *functions* inside the service are AWS Lambda functions.

## Configuration

All of the Lambda functions in your serverless service can be found in `serverless.yml` under the `functions` property.

```yml
# serverless.yml
service: myService

provider:
  name: aws
  runtime: nodejs6.10
  memorySize: 512 # optional, default is 1024
  timeout: 10 # optional, default is 6
  versionFunctions: false # optional, default is true

functions:
  hello:
    handler: handler.hello # required, handler set in AWS Lambda
    name: ${self:provider.stage}-lambdaName # optional, Deployed Lambda name
    description: Description of what the lambda function does # optional, Description to publish to AWS
    runtime: python2.7 # optional overwrite, default is provider runtime
    memorySize: 512 # optional, default is 1024
    timeout: 10 # optional, default is 6
```

The `handler` property points to the file and module containing the code you want to run in your function.

```javascript
// handler.js
module.exports.functionOne = function(event, context, callback) {}
```

You can add as many functions as you want within this property.

```yml
# serverless.yml

service: myService

provider:
  name: aws
  runtime: nodejs6.10

functions:
  functionOne:
    handler: handler.functionOne
    description: optional description for your Lambda
  functionTwo:
    handler: handler.functionTwo
  functionThree:
    handler: handler.functionThree
```

Your functions can either inherit their settings from the `provider` property.

```yml
# serverless.yml
service: myService

provider:
  name: aws
  runtime: nodejs6.10
  memorySize: 512 # will be inherited by all functions

functions:
  functionOne:
    handler: handler.functionOne
```

Or you can specify properties at the function level.

```yml
# serverless.yml
service: myService

provider:
  name: aws
  runtime: nodejs6.10

functions:
  functionOne:
    handler: handler.functionOne
    memorySize: 512 # function specific
```

## Permissions

Every AWS Lambda function needs permission to interact with other AWS infrastructure resources within your account.  These permissions are set via an AWS IAM Role.  You can set permission policy statements within this role via the `provider.iamRoleStatements` property.

```yml
# serverless.yml
service: myService

provider:
  name: aws
  runtime: nodejs6.10
  iamRoleStatements: # permissions for all of your functions can be set here
    - Effect: Allow
      Action: # Gives permission to DynamoDB tables in a specific region
        - dynamodb:DescribeTable
        - dynamodb:Query
        - dynamodb:Scan
        - dynamodb:GetItem
        - dynamodb:PutItem
        - dynamodb:UpdateItem
        - dynamodb:DeleteItem
      Resource: "arn:aws:dynamodb:us-east-1:*:*"

functions:
  functionOne:
    handler: handler.functionOne
    memorySize: 512
```

Another example:

```yml
# serverless.yml
service: myService
provider:
  name: aws
  iamRoleStatements:
      -  Effect: "Allow"
         Action:
           - "s3:ListBucket"
         Resource: { "Fn::Join" : ["", ["arn:aws:s3:::", { "Ref" : "ServerlessDeploymentBucket"} ] ] } # You can put CloudFormation syntax in here.  No one will judge you.  Remember, this all gets translated to CloudFormation.
      -  Effect: "Allow"
         Action:
           - "s3:PutObject"
         Resource:
           Fn::Join:
             - ""
             - - "arn:aws:s3:::"
               - "Ref" : "ServerlessDeploymentBucket"
               - "/*"

functions:
  functionOne:
    handler: handler.functionOne
    memorySize: 512
```

You can also use an existing IAM role by adding your IAM Role ARN in the `role` property. For example:

```yml
# serverless.yml
service: new-service
provider:
  name: aws
  role: arn:aws:iam::YourAccountNumber:role/YourIamRole
```

See the documentation about [IAM](./iam.md) for function level IAM roles.

## VPC Configuration

You can add VPC configuration to a specific function in `serverless.yml` by adding a `vpc` object property in the function configuration. This object should contain the `securityGroupIds` and `subnetIds` array properties needed to construct VPC for this function. Here's an example configuration:

```yml
# serverless.yml
service: service-name
provider: aws

functions:
  hello:
    handler: handler.hello
    vpc:
      securityGroupIds:
        - securityGroupId1
        - securityGroupId2
      subnetIds:
        - subnetId1
        - subnetId2
```

Or if you want to apply VPC configuration to all functions in your service, you can add the configuration to the higher level `provider` object, and overwrite these service level config at the function level. For example:

```yml
# serverless.yml
service: service-name
provider:
  name: aws
  vpc:
    securityGroupIds:
      - securityGroupId1
      - securityGroupId2
    subnetIds:
      - subnetId1
      - subnetId2

functions:
  hello: # this function will overwrite the service level vpc config above
    handler: handler.hello
    vpc:
      securityGroupIds:
        - securityGroupId1
        - securityGroupId2
      subnetIds:
        - subnetId1
        - subnetId2
  users: # this function will inherit the service level vpc config above
    handler: handler.users
```

Then, when you run `serverless deploy`, VPC configuration will be deployed along with your lambda function.

**VPC IAM permissions**

The Lambda function execution role must have permissions to create, describe and delete [Elastic Network Interfaces](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_ElasticNetworkInterfaces.html) (ENI). When VPC configuration is provided the default AWS `AWSLambdaVPCAccessExecutionRole` will be associated with your Lambda execution role. In case custom roles are provided be sure to include the proper [ManagedPolicyArns](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-iam-role.html#cfn-iam-role-managepolicyarns). For more information please check [configuring a Lambda Function for Amazon VPC Access](http://docs.aws.amazon.com/lambda/latest/dg/vpc.html)

## Environment Variables

You can add environment variable configuration to a specific function in `serverless.yml` by adding an `environment` object property in the function configuration. This object should contain a key/value collection of strings:

```yml
# serverless.yml
service: service-name
provider: aws

functions:
  hello:
    handler: handler.hello
    environment:
      TABLE_NAME: tableName
```

Or if you want to apply environment variable configuration to all functions in your service, you can add the configuration to the higher level `provider` object. Environment variables configured at the function level are merged with those at the provider level, so your function with specific environment variables will also have access to the environment variables defined at the provider level. If an environment variable with the same key is defined at both the function and provider levels, the function-specific value overrides the provider-level default value. For example:

```yml
# serverless.yml
service: service-name
provider:
  name: aws
  environment:
    SYSTEM_NAME: mySystem
    TABLE_NAME: tableName1

functions:
  hello:
    # this function will have SYSTEM_NAME=mySystem and TABLE_NAME=tableName1 from the provider-level environment config above
    handler: handler.hello
  users:
    # this function will have SYSTEM_NAME=mySystem from the provider-level environment config above
    # but TABLE_NAME will be tableName2 because this more specific config will override the default above
    handler: handler.users
    environment:
      TABLE_NAME: tableName2
```

## Tags

Using the `tags` configuration makes it opssible to add `key` / `value` tags to your functions.

Those tags will appear in your AWS console and makes it easier for you to group functions by tag or find functions with a common tag.

```yml
functions:
  hello:
    handler: handler.hello
    tags:
      foo: bar
```

Real-world use cases where tagging your functions is helpful include:

- Cost estimations (tag functions with an environemnt tag: `environment: Production`)
- Keeping track of legacy code (e.g. tag functions which use outdated runtimes: `runtime: nodejs0.10`)
- ...

## Log Group Resources

By default, the framework will create LogGroups for your Lambdas. This makes it easy to clean up your log groups in the case you remove your service, and make the lambda IAM permissions much more specific and secure.


## Versioning Deployed Functions

By default, the framework creates function versions for every deploy. This behavior is optional, and can be turned off in cases where you don't invoke past versions by their qualifier. If you would like to do this, you can invoke your functions as `arn:aws:lambda:....:function/myFunc:3` to invoke version 3 for example.

To turn off this feature, set the provider-level option `versionFunctions`.

```yml
provider:
  versionFunctions: false
```

These versions are not cleaned up by serverless, so make sure you use a plugin or other tool to prune sufficiently old versions. The framework can't clean up versions because it doesn't have information about whether older versions are invoked or not. This feature adds to the number of total stack outputs and resources because a function version is a separate resource from the function it refers to.
