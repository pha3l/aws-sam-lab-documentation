# Authoring the Vote Function
We now have a way to create new polls, retrieve the data for a specific poll, and list all of the created polls.  For the sake of brevity, we'll forgo the other usual CRUD operations and move on to vote-casting.

As we've been doing, let's add another method to our `DynamoRepository` abstraction.  Remember that this is a *generic* implementation that is designed to be useful for any type that we'd like to persist to DynamoDB.  With this in mind, create an `update` method in the class:
```typescript
public update(id: string, updateExpression: UpdateExpression, expressionAttributeValues: DynamoDB.DocumentClient.ExpressionAttributeValueMap): Promise<UpdateItemOutput> {
    let params = {
        TableName: this.tableName,
        Key: {
            id: id
        },
        UpdateExpression: updateExpression,
        ExpressionAttributeValues: expressionAttributeValues,
        ReturnValues: "ALL_NEW"
    } as UpdateItemInput;

    return new Promise<UpdateItemOutput>((resolve, reject) => {
        this.dynamoClient.update(params, (err, data) => {
            if (err) reject(err);
            else resolve(data);
        });
    });
}
```

Create the `src/vote.function.ts` file:
```typescript
import { Handler, APIGatewayEvent, Context, Callback } from 'aws-lambda';
import { Poll } from './poll.model';
import { DynamoRepository } from './dynamo-repository';
import { ApiGatewayResponse, ResponseType } from './api-gateway-response';
import { DocumentClient } from 'aws-sdk/lib/dynamodb/document_client';
import { DynamoDB } from 'aws-sdk';
import { UpdateItemInput } from 'aws-sdk/clients/dynamodb';

export const handler: Handler = (event: APIGatewayEvent, context: Context, callback?: Callback) => {
    const pollId = event.pathParameters.pollId;
    const answerIndex = event.pathParameters.answerIndex;

    const dynamoClient: DocumentClient = new DynamoDB.DocumentClient;

    let params = {
        TableName: process.env.TABLE_NAME,
        Key: {
            id: pollId
        },
        UpdateExpression: "SET answers[" + answerIndex + "].votes = answers[" + answerIndex + "].votes + :one",
        ExpressionAttributeValues: {
           ':one' : 1
        },
        ReturnValues: "ALL_NEW"
    } as UpdateItemInput;

    dynamoClient.update(params).promise().then(data => {
        callback(null, new ApiGatewayResponse(ResponseType.OK, data));
    }, error => {
        callback(null, new ApiGatewayResponse(ResponseType.SERVER_ERROR, error));
    });
}
```
Now, this is a little more involved than the rest of our functions so far, so let's go through it quickly.  First, we're getting two parameters from the route path, the poll the question being voted for belongs to (`pollId`) and the index of the question in the `answers` list of that poll (`answerIndex`).  We're then constructing the `UpdateItemInput` object for the `DocumentClient#update` method, first defining the usual information (table name, primary key of object we want to update).  Then, we define an `UpdateExpression` parameter, which is the Dynamo equivalent of a `SET` clause in a SQL `UPDATE` statement.  In the expression, we're setting the `votes` attribute of the answer at the passed in index to it's current value plus the value of an `ExpressionAttribute` called `:one`.  Finally, we give the value for `:one` as `1`, and tell Dynamo we'd like to get back the entire updated object after the atomic operation.  What we've accomplished with this is updating the counter in place.  Hopefully this simple example is a decent illustration of the  power of the DynamoDB `update` action. There are further parameters available that allow the attribute(s) to be modified to be set programmatically as well!

Let's add the function to our `template.yml`:
```yaml
CastVote:
    Type: AWS::Serverless::Function
    Properties:
        CodeUri: ./dist/vote/vote.zip
        Handler: index.handler
        Runtime: nodejs6.10
        Policies: AmazonDynamoDBFullAccess
        Events:
            CreatePoll:
                Type: Api
                Properties:
                    Path: /polls/{pollId}/answer/{answerIndex}
                    Method: PATCH
            Environment:
                Variables:
                    TABLE_NAME: !Ref Polls
```
Note that we're using a PATCH request for this operation, which is the standard for partial updates on a resource.  Also note that we've set the path to have both of the path parameters we used in the function.

All that's left to do is a quick test.  Generate the mock payload (make sure you replace the `<pollId>` with your id):
```bash
$ sam local generate-event api --method patch --resource /polls/{pollId}/answer/{answerIndex} \
  --path /polls/<pollId>/answers/0 > mocks/vote.json
```

Edit it once again to fix the path parameters:
```json
"pathParameters": {
    "proxy": "/polls/c2a7ca91-a598-4941-8f2e-5dda85482cb9/answers/0"
}
```
Should become:
```json
"pathParameters": {
    "pollId": "c2a7ca91-a598-4941-8f2e-5dda85482cb9",
    "answerIndex": "0"
}
```
`webpack`, then invoke the function:
```bash
$ AWS_REGION="us-west-2" TABLE_NAME="poll-testdb" sam local invoke CastVote -e mocks/vote.json
```

Now let's check the record:
```bash
$ aws --region us-west-2 dynamodb scan --table-name poll-testdb 
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
                                "N": "1"
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
The vote count for the first answer, Black Bear, is now 1! Hooray!

We're done authoring the backend of our application.  Let's wrap up a couple loose ends in the CloudFormation template and get it deployed in the last part of this section.