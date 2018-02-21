# sam-local
SAM Local is a tool (currently in beta) provided by AWS that allows testing your SAM applications locally on your dev machine.  It also has some nice utilities for helping to create test payloads for your functions. It uses Docker under the hood, so make sure that you have Docker installed first ([link](https://docs.docker.com/docker-for-mac/install/)). Luckily for us, SAM Local is distributed as an npm package.

```bash
$ npm install -g aws-sam-local
```

For more information on SAM Local, check out:
* [https://github.com/awslabs/aws-sam-local](https://github.com/awslabs/aws-sam-local)
* [https://docs.aws.amazon.com/lambda/latest/dg/test-sam-local.html](https://docs.aws.amazon.com/lambda/latest/dg/test-sam-local.html)