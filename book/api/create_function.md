# Authoring the Create Function
With Typescript, we can use an interface to define the structure and expected properties (and their types) of an object, and use this to our advantage as we write our application to provide type-checking.  We'll use this to ensure we have a consistent data model across all of our functions.  Let's start by defining an interface, `poll.model.ts`, at the root of the `src` folder:

`src/poll.model.ts`
```typescript
export interface Poll {
    id: string;
    title: string;
    question: string;
    answers: Array<string>;
    createdAt: number;
    updatedAt: number;
}
```

But we also need a way to track the number of votes for each answer.  In the same file below the `Poll` interface definition, create another interface, `PollAnswer` as follows:
```typescript
export interface PollAnswer {
    answer: string;
    votes: number;
}
```
And then change the type of the `answers` property to `Array<PollAnswer>`.

The final file should look like this now:
```typescript
export interface Poll {
    id: string;
    title: string;
    question: string;
    answers: Array<PollAnswer>;
    createdAt: number;
    updatedAt: number;
}

export interface PollAnswer {
    answer: string;
    votes: number;
}
```

Now, we want to avoid duplicating code between our functions so let's implement a basic repository design pattern that will abstract the DynamoDB logic.  With only one table/model, this may seem like overkill, but in a more complex project it's a great way to DRY up the persistence logic.  Create a file `dynamo-repository.ts`:

`src/dynamo-repository.ts`
```typescript
import { DocumentClient } from 'aws-sdk/lib/dynamodb/document_client';
import { DynamoDB } from 'aws-sdk';
import { PutItemInput, PutItemOutput } from 'aws-sdk/clients/dynamodb';

export class DynamoRepository<T> {
    private dynamoClient : DocumentClient;

    constructor(private tableName: string) {
        if (tableName.match(/.*-test$/g)) {
            console.log("Using local Dynamo table: " + tableName);
            this.dynamoClient = new DynamoDB.DocumentClient({
                endpoint: 'http://172.16.123.1:8000'
            });
        } else {
            this.dynamoClient = new DynamoDB.DocumentClient();
        }
    }

    public create(entity: T) : Promise<PutItemOutput> {
        let params = {
            TableName: this.tableName,
            Item: entity as Object
        } as PutItemInput;

        return new Promise<PutItemOutput>((resolve, reject) => {
            this.dynamoClient.put(params, (err, data) => {
                if (err) reject(err);
                else resolve(data);
            });
        });
    }

}
```
What we've done here is create a generic repository class that will work with any table/entity.  Note the constructor--we've set it up to check the name of the table against a regular expression looking for a `-test` suffix.  If it's there, we'll instantiate the `DocumentClient` with a local endpoint.  We're using this sort of strange suffix strategy to prevent introducing an additional environment variable because for `sam-local` to pick up environment variables, they have to be defined in the template, and we don't really want to add one to the production environment just so our tests will run properly.  All this will allow us to use a local Dynamo table for testing.  The IP we've used, `172.16.123.1`, is a random IP in the private IP space.  The reason that we're not just using `localhost` here is because when we test our functions with `sam local`, the functions will run in a docker container, while the Dynamo Local endpoint will be on localhost.  Therefore we'll have to set up an alias so the function can find localhost a little later.

Along the same lines, let's create a class to represent the response that API Gateway expects when invoking the lambda on the user's behalf.  Setting the `Access-Control-Allow-Origin` header is **VERY IMPORTANT**, as we'll be interacting with the API via AJAX calls, and thus CORS will be an issue. Create a file `api-gateway-response.ts`:

`src/api-gateway-response.ts`
```typescript
export class ApiGatewayResponse {
    private statusCode: number;
    private headers: {};
    private body: string;
    private isBase64Encoded: boolean;

    constructor(responseType: ResponseType, body: Object, isBase64Encoded?: boolean) {
        this.statusCode = responseType;
        this.body = JSON.stringify(body);
        this.isBase64Encoded = isBase64Encoded || false;
        this.headers = {
            "Access-Control-Allow-Origin": "*"
        };
    }   
}

export enum ResponseType {
    OK = 200,
    SERVER_ERROR = 500,
    NOT_FOUND = 404
}
```

Unlike relational database systems that you may be more familiar with, DynamoDB and other object-based NoSQL databases don't have a utility for generating unique identifiers built in, so let's add an NPM package to help us generate them in our function:

```
$ npm install --save uuid
```

We installed the `uuid` package with the `--save` flag, which means that it will not only be installed to `node_modules`, but will be added to our `package.json`.

While we're on the subject of npm dependencies, you might have noticed that the `aws-sdk` is specified as a dev dependency.  This is because lambda provides this in the execution environment, so we don't need to bundle it, which is great because it's a large package.

Now we have all the pieces in place, and we can implement the actual function.  Create the file `create-poll.function.ts`:

`src/create-poll.function.ts`
```typescript
import { Handler, APIGatewayEvent, Context, Callback } from 'aws-lambda';
import { v4 as uuid } from 'uuid';
import { Poll } from './poll.model';
import { DynamoRepository } from './dynamo-repository';
import { ApiGatewayResponse, ResponseType } from './api-gateway-response';

export const handler: Handler = (event: APIGatewayEvent, context: Context, callback?: Callback) => {
    let poll = JSON.parse(event.body) as Poll;
    let now = new Date().getTime();

    poll.createdAt = now;
    poll.updatedAt = now;
    poll.id = uuid();

    new DynamoRepository<Poll>(process.env.TABLE_NAME).create(poll).then((val) => {
        callback(null, new ApiGatewayResponse(ResponseType.OK, poll));
    }).catch((err) => {
        callback(null, new ApiGatewayResponse(ResponseType.SERVER_ERROR, { status: false, error: err }));
    });
}
```

The last step is to describe our function within the CloudFormation template.  Remove the section for the HelloWorldSample function and add the following under the Dynamo table in `template.yml`:

`template.yml`
```yaml
...
CreatePoll:
    Type: AWS::Serverless::Function
    Properties:
        CodeUri: ../dist/create-poll/create-poll.zip
        Handler: index.handler
        Runtime: nodejs6.10
        Policies: AmazonDynamoDBFullAccess
        Events:
            CreatePoll:
                Type: Api
                Properties:
                Path: /polls
                Method: POST
        Environment:
            Variables:
                TABLE_NAME: !Ref Polls
```

Here you can see we've added a new function with a **Logical ID** of `CreatePoll`, and pointed it at the transpiled, packaged, and minified webpack output for the function script.  We've also given the function a managed policy, `AmazonDynamoDBFullAccess`, that gives it full access to DynamoDB resources.  While this isn't the best idea for production use and you'd want to define a role with more limited scope (which you can do right in the CloudFormation template and reference directly!), it will be fine for our purposes. We've also defined the event triggers for the function, in this case an API Gateway route of `/polls` with a `POST` HTTP method.  Finally, we've given the function access to an environment variable called `TABLE_NAME`, which we already used in the function code to fetch the table name. We've then used the !Ref intrinsic function to obtain the table name from the resource defined above.  This is necessary because while we've given the table a *logical name* of `Polls`, the physical name will be a generated value consisting of the stack name, the logical name, and a random identifier at deploy time.

===

With our first function authored, let's do a quick test to see how we're doing so far. Let's generate a mock payload using `sam-local`:

```bash
$ sam local generate-event api -m POST -b "{\\\"title\\\":\\\"Test Poll\\\", \\\"question\\\": \\\"Which bear is best?\\\", \\\"answers\\\": [{\\\"answer\\\":\\\"Black Bear\\\", \\\"votes\\\":0},{\\\"answer\\\":\\\"That's debatable...\\\", \\\"votes\\\":0}]}" > mocks/create-poll.json
``` 

Now, we'll need a local DynamoDB table to use for testing.  To start up the local Dynamo server that we downloaded earlier, navigate to the directory you extracted it to and run the following (do this in a separate terminal as we want to keep it running from now on):
```bash
$ java -Djava.library.path=./DynamoDBLocal_lib -jar DynamoDBLocal.jar -sharedDb
```

 let's create a table to use for our testing. We'll do so using the CLI:
```bash
$ aws dynamodb create-table --region us-west-2 \
                            --table-name poll-testdb \
                            --attribute-definitions AttributeName=id,AttributeType=S \
                            --key-schema AttributeName=id,KeyType=HASH \
                            --provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5 \
                            --endpoint-url http://localhost:8000
```
Note that this is the exact same CLI command you'd use to provision a real DynamoDB table, with the simple addition of an `endpoint-url` parameter to redirect the request to the local server.

Earlier we talked about setting up an alias so the container-bound function can find the local mock-Dynamo server.  Let's do that now:
```
$ sudo ifconfig lo0 alias 172.16.123.1
```

Run `webpack` to generate the function package:
```bash
$ webpack
```

Now let's invoke the function using `sam-local`, sending our mock event as the payload:
```bash
$ TABLE_NAME="poll-test" sam local invoke CreatePoll -e mocks/create-poll.json
```
Note that we passed the environment variables needed by the function directly to `sam-local`.  Even if they are defined in the CloudFormation template, `sam-local` will not pick them up so this is necessary.

If everything went okay, you should see `{"statusCode":200,"body":{}}` at the bottom of the output.  You can use the CLI to check:
```bash
$ aws dynamodb scan --table-name poll-test --endpoint http://localhost:8000
{
    "Items": [
        {
            "createdAt": {
                "N": "1518585027076"
            },
            "question": {
                "S": "Which bear is best?"
            },
            "answers": {
                "L": [
                    {
                        "M": {
                            "answer": {
                                "S": "Black Bear"
                            },
                            "votes": {
                                "N": "0"
                            }
                        }
                    },
                    {
                        "M": {
                            "answer": {
                                "S": "That's debatable..."
                            },
                            "votes": {
                                "N": "0"
                            }
                        }
                    }
                ]
            },
            "id": {
                "S": "c2a7ca91-a598-4941-8f2e-5dda85482cb9"
            },
            "updatedAt": {
                "N": "1518585027076"
            },
            "title": {
                "S": "Test Poll"
            }
        }
    ],
    "Count": 1,
    "ScannedCount": 1,
    "ConsumedCapacity": null
}
```

---

As you can see, we successfully created a record in the table! Hold on to the id of the returned record, we'll use it for further tests.  Let's continue fleshing out the API functionality. In the next section, we'll create a function to retrieve a single poll record by it's id.
