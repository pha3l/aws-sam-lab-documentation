# API Deployment
We're done writing our Lambdas and we're ready to deploy our SAM application.  The `sam-local` package we've been using to test our functions locally has some helpful utilities for deployment, so we'll take advantage of these.

Our functions access two environment variables at runtime, so we need to make sure both of these are available to the lambda runtime.  We have already provided the `TABLE_NAME` using the `!Ref` intrinsic function to each function separately, but let's specify `AWS_REGION` globally.  To do this, we'll need to add the same structure we used in the function resources to the top level of our yaml structure, let's put it right below the `Description` parameter:
```yaml
AWSTemplateFormatVersion: 2010-09-09
Transform: AWS::Serverless-2016-10-31
Description: A serverless poll app
Environment:
  Variables:
    AWS_REGION: !Ref AWS::Region
```
We've used the `AWS::Region` pseudo-parameter along with the `!Ref` intrinsic function to get the region that the template is being deployed in.

Now, we can ensure our template is syntactically correct by running:
```bash
$ sam validate
```
If there are no errors, the output should be `Valid!`.  Note that this does not ensure that our template will provision without issues, but simply that its syntax is valid.

We're now just about ready to deploy, but we need *one more thing*.  Since we've used local path references to our code in the template, if we deployed the template as-is CloudFormation would have no way of accessing the code.  Luckily, CloudFormation provides a `package` operation that is designed to take care of this problem.  This takes a template file, uploads all of the referenced files to an S3 bucket, and outputs a new template that references the S3 objects instead of the local paths.  To take advantage of this, we'll need to provision an S3 bucket for it to use to store the code.  You can do this via the console or the CLI:
**The region you use here must be the same region you'd like to deploy the application to.**
```bash
$ aws s3api create-bucket --bucket my-poll-app-cfn --region us-west-2 \
    --create-bucket-configuration LocationConstraint=us-west-2
```
I've used `my-poll-app-cfn`, but remember that S3 buckets are globally unique, so you'll have to choose something else for your bucket name.  Also note that for this particular CLI call, any region outside of us-east-1 requires the `LocationConstraint` parameter passed to `--create-bucket-configuration`.  

Run the package command, specifying the bucket we just created:
```bash
$ sam package --template-file template.yml --s3-bucket my-poll-app-cfn --output-template-file template-packaged.yml
```
Note that you can also use `aws cloudformation` instead of `sam` as the command with the same parameters to get the same result.

Let's take a quick look at the `template-packaged.yml` file that was generated. Here's a snippet containing our `CastVote` resource:
```yaml
CastVote:
    Properties:
      CodeUri: s3://my-poll-app-cfn/51d31d05f68d1c495f3e25f5614d8f5d
      Environment:
        Variables:
          TABLE_NAME:
            Ref: Polls
      Events:
        CreatePoll:
          Properties:
            Method: PATCH
            Path: /polls/{pollId}/answer/{answerIndex}
          Type: Api
      Handler: index.handler
      Policies: AmazonDynamoDBFullAccess
      Runtime: nodejs6.10
    Type: AWS::Serverless::Function
```
Our local path in the `CodeUri` property has been replaced with an S3 object path.

---

The time has finally come.  Let's deploy the stack:
```bash

```

If there is a problem with the stack creation, you can remove the failed stack, which will be stuck in a `ROLLBACK_COMPLETE` state and thus won't let you redeploy the same stack, with the following:
```bash
$ aws cloudformation delete-stack --stack-name my-poll-app-api --region us-west-2
```