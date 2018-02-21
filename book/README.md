# Building Serverless Applications on AWS with SAM and Typescript
#### By Luke Merry
## Introduction

In this hands-on training lab, we will be exploring the relatively new concept of "Serverless" web applications.  What does this mean? A serverless application takes advantage of a cloud computing offering known generically as Function as a Service, or FaaS.  In comparison to classic Infrastructure as a Service, which typically provides services like virtual machines and managed databases, FaaS provides a high level of abstraction from infrastructure and allows you to simply write code that runs in response to events, without ever thinking about typical DevOps considerations like keeping the OS patched, scaling the application, or recovering from hardware or software failure.  You simply write your code and define the conditions under which it will run, which could be an HTTP request, a message published to a pub/sub notification system such as SNS or SQS, or a file being uploaded to an S3 bucket.

We'll be building a simple web app using a completely serverless approach.  Our application will allow users to create polls with a question and several answers, and tally votes for each answer.  While this is a trivial application, the concepts covered here can be applied to problems of any complexity.  This lab is designed to introduce basic concepts, design patterns, and best practices for building a serverless application, and provide a foundation upon which you can explore the idea further.  

### Benefits of Serverless Over Traditional Web Applications
The benefits of building a serverless application are many, but there are three primary advantages that come to mind:

#### Infrastructure Cost Savings
In a traditional application, it is necessary to always maintain a minimum threshold of compute capacity.  When your application is idling, you still have to pay for those resources which allow your application to respond when the requests do start flowing.
Serverless architectures allow for your application to only cost you money for the precise moments that it's doing something.  So if, for instance, it takes 50 milliseconds for your app to fetch a list of Widgets that your user might be interested in purchasing, you pay for only this *exact* compute time (rounded to the nearest 100ms in the case of Lambda).

#### Virtually Unlimited Scalability
AWS Lambda automatically scales to serve simultaneous requests.  That means that if your function is busy serving a request when another arrives, AWS will automatically provision another copy of the function to serve the new request.  The same goes for 10,000 or 1,000,000 simultaneous requests.  It's important to keep in mind that there is a slight performance hit for requests that require a new function instance to be provisioned (known as a cold start), but there are a variety of ways to mitigate this issue.  The best step that you can take to minimize the impact of cold starts is to choose a language with minimal overhead.  Our choices on AWS consist of Python, Node.js, Java, C#/.NET Core, and a very recent addition, Golang.  Of these, the options with the fastest cold starts are Python and Javascript, and I've chosen to use Node.js for this training.  Specifically, we'll be using Typescript to author our functions, and we'll use webpack and babel to transpile it to optimized, tree-shaken, and minified javascript.

#### Development Velocity
Being able to focus on a single operation at a time when developing can help improve velocity substantially.  That's not to say that Lambda functions cannot take advantage of shared code or apply DRY principles, and we'll be illustrating how to do this throughout the training.

---

### Application Overview
The application consists of several Lambda functions authored in Typescript that will make up the backend, and an Angular SPA frontend.  Data will be persisted by DynamoDB, and for the sake of simplicity we'll forgo the notion of users and authentication, but I have an expanded training in the works involving authentication with Amazon Cognito.

We will be taking advantage of a model known as AWS SAM, or Serverless Application Model.  This is not a "framework" in the traditional sense, but more of a set of principles and best practices for building serverless apps with the tool chain provided by AWS.  It does, however, include a CloudFormation "Transform" which allows for a simplified CloudFormation template syntax that is specifically designed for describing serverless applications, which we will be covering in depth.  In practice, this means that our CloudFormation template will be pre-processed by an additional processing layer, which will expand our SAM syntax into full CloudFormation syntax.  You can see this process transparently by viewing the deployed CloudFormation template within the AWS Console.

I chose SAM over the alternative [Serverless Framework](https://serverless.com) for this lab because while the latter is probably easier to get started with, I feel that SAM is just the right amount of abstraction and allows for very fine-grained control over the application stack, as well as providing the full force of CloudFormation functionality available within the same template.  This allows one to specify other resources to be provisioned along with the serverless application, and thus the entire application stack/infrastructure can be managed as code in a single construct.  Serverless Framework is a very solid choice, however, and I encourage you to try it out and compare the two.

This lab is divided into two main sections:
- [API](./api/intro.md)
- [Client App](./client/intro.md)

The first section will focus on setting up the resources we will use to persist data within the application, and authoring the Lambda functions that will back our API. In the second section, we will build a simple client application to interact with our API utilizing Angular 5 and components from Google's Material Design.  At the end of the lab, you'll have built a completely serverless application utilizing many of the awesome developer-centric services provided by AWS, including:

- Lambda
- API Gateway
- DynamoDB
- CloudFormation
- S3

So without further ado, let's get started!
