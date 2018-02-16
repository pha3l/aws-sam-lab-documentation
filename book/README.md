# Introduction

In this lab, we will be building a simple polling app using a completely serverless approach.  While this is a trivial application, the concepts covered here can be applied to problems of any complexity.  This lab is designed to introduce the concepts, design patterns, and best practices for building a serverless application, and provide a foundation upon which you can explore the idea further.  

---

The benefits of building a serverless application are many, but there are three primary advantages that come to mind:
#### Infrastructure Cost Savings
In a traditional application, it is necessary to always maintain a minimum threshold of compute capacity.  When your application is idling, you still have to pay for those resources which allow your application to respond when the requests do start flowing.
Serverless architectures allow for your application to only cost you money for the precise moments that it's doing something.  So if, for instance, it takes 50 milliseconds for your app to fetch a list of Widgets that your user might be interested in purchasing, you pay for only this compute time, rounded to the nearest 100ms in the case of Lambda.
#### Virtually Unlimited Scalability
AWS Lambda automatically scales to serve simultaneous requests.  That means that if your function is busy serving a request when another arrives, AWS will automatically provision another copy of the function to serve the new request.  Same goes for 10,000 or 1,000,000 simultaneous requests.  There is a slight performance hit for requests that require a new function instance to be served (known as a cold start), but there are a variety of ways to mitigate this issue.  The best step that you can take to minimize the impact of cold starts is to choose a language with minimal overhead.  In the case of Lambda, the best choices for an API scenario then become Javascript or Python, which is why we've gone with Javascript here.
#### Development Velocity
Being able to focus on a single operation at a time when developing can help improve velocity substantially.  That's not to say that Lambda functions cannot take advantage of shared code or apply DRY principles.

---

The application consists of several Lambda functions authored in Typescript that will make up the backend, and an Angular SPA frontend.  Data will be persisted by DynamoDB, and for the sake of simplicity we'll forgo the notion of users and authentication, but look for another tutorial soon involving authentication with Amazon Cognito.

We will be taking advantage of a model known as AWS SAM, or Serverless Application Model.  This is not a "framework" in the traditional sense, but more of a set of principles and best practices for building serverless apps with the toolset provided by AWS.  It does, however, include a CloudFormation "Transform" which allows for a simplified CloudFormation template syntax that is specifically designed for describing serverless applications, which we will be covering in depth.  In practice, this means that our CloudFormation template will be preprocessed by an additional processing layer, which will expand our SAM syntax into full CloudFormation syntax.  You can see this process transparently by viewing the deployed CloudFormation template within the AWS Console.

I chose SAM over the alternative [Serverless Framework](https://serverless.com) for this lab because while the latter is probably easier to get started with, I feel that SAM is just the right amount of abstraction and allows for very fine-grained control over the application stack, as well as providing the full force of CloudFormation functionality available within the same template.  This allows one to specify other resources to be provisioned along with the serverless application, and thus the entire application stack/infrastructure can be managed as code in a single construct.

This lab is divided into two main sections:
- [API](./api/intro.md)
- [Client App](./client/intro.md)

The first section will focus on setting up the resources we will use to persist data within the application, and authoring the Lambda functions that will back our API. In the second section, we will build a simple client application to interact with our API utilizing Angular 5 and components from Google's Material Design.  At the end of the lab, you'll have built a completely serverless application utilizing many of the awesome developer-centric services provided by AWS, including:

- Lambda
- API Gateway
- DynamoDB
- CloudFormation/SAM

So without further ado, let's get started!
