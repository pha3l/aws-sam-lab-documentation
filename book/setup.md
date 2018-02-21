# Development Environment Setup
There are a few things you should have set up on your machine before moving forward with the lab.

## AWS CLI
The easiest way to get the CLI installed is via pip, the python package manager:
```bash
$ pip install awscli --upgrade --user
```

After installation, you'll need to configure the CLI to use your IAM credentials:
```bash
$ aws configure
```

Enter your credentials as prompted.  For default region and output format, you can just leave them empty.
```bash
AWS Access Key ID [None]: AKIA******************** 
AWS Secret Access Key [None]: **************************** 
Default region name [None]: 
Default output format [None]: 
```

## Node.js/npm
As we'll be using Typescript to author the lambda functions, as well as Angular for the client, you'll need Node.js and npm.  Homebrew is easiest for mac, but there are instructions for other platforms [here](https://www.npmjs.com/get-npm).
```bash
$ brew install node
```
While you're at it, install the Angular CLI which we'll need a little later:
```bash
$ npm install -g @angular/cli
```

## Yeoman
To save some time, I've created a *Yeoman Generator* for the backend API project.  Like the Angular CLI, this will help us by initializing a project for the SAM application, saving us a lot of manual configuration.

You'll need to install Yeoman itself, as well as the generator:
```bash
$ npm install -g yo
$ npm install -g generator-aws-sam-typescript
```

# sam-local
SAM Local is a tool (currently in beta) provided by AWS that allows testing your SAM applications locally on your dev machine.  It also has some nice utilities for helping to create test payloads for your functions. It uses Docker under the hood, so make sure that you have Docker installed first ([link](https://docs.docker.com/docker-for-mac/install/)). Luckily for us, SAM Local is distributed as an npm package, so it's quite easy to install.

```bash
$ npm install -g aws-sam-local
```

For more information on SAM Local, check out:
* [https://github.com/awslabs/aws-sam-local](https://github.com/awslabs/aws-sam-local)
* [https://docs.aws.amazon.com/lambda/latest/dg/test-sam-local.html](https://docs.aws.amazon.com/lambda/latest/dg/test-sam-local.html)

### DynamoDB Local
If you want to be able to test the data persistence locally, there is a great resource that allows you to run a DynamoDB server on your development machine.  Note that it is a Java implementation, so you'll also need to have Java 8 installed on your machine before you can run it.  Also worth noting is that this is only intended for testing and local development and should under no circumstances be used for production applications. You can download it [here](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DynamoDBLocal.html).  Just extract the archive somewhere for now, and remember where you put it.

### Visual Studio Code
Other editors will work fine, obviously, but since we'll be working with Typescript, I highly recommend VSCode for the language support which, as far as I've seen, is unparalleled.  Get it [here](https://code.visualstudio.com/download).
