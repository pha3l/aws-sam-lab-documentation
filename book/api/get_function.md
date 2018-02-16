# Authoring the Get Function
We already have a lot of the building blocks in place, so we can dive right in.  First let's modify the `DynamoRepository` class by adding a `get` method:
```typescript
public get(id: string) : Promise<GetItemOutput> {
    let params = {
        TableName: this.tableName,
        Key: {
            id: id
        }
    } as GetItemInput;

    return new Promise<GetItemOutput>((resolve, reject) => {
        this.dynamoClient.get(params, (err, data) => {
            if (err) reject(err);
            else resolve(data);
        });
    });
}
```
As you can see, this is very similar to the pattern we used for the create method.  We're returning a `Promise` which should resolve to contain the fetched item.

Now let's create the actual function.  Create a new file `get-poll.function.ts` at the root of the `src` directory and populate it as follows:
```typescript
import { Handler, APIGatewayEvent, Context, Callback } from 'aws-lambda';
import { Poll } from './poll.model';
import { DynamoRepository } from './dynamo-repository';
import { ApiGatewayResponse, ResponseType } from './api-gateway-response';

export const handler: Handler = (event: APIGatewayEvent, context: Context, callback?: Callback) => {
    const id = event.pathParameters.id;
    
    new DynamoRepository<Poll>(process.env.TABLE_NAME).get(id).then((val) => {
        if (!val.Item)
            callback(null, new ApiGatewayResponse(ResponseType.NOT_FOUND, null));
        callback(null, new ApiGatewayResponse(ResponseType.OK, val.Item));
    }).catch((err) => {
        callback(null, 
            new ApiGatewayResponse(ResponseType.SERVER_ERROR, { status: false, error: err }));
    });
}
```
We've added a new type to the `ResponseType` enumeration, so head over to the `api-gateway-response.ts` file and add it:
```typescript
export enum ResponseType {
    OK = 200,
    SERVER_ERROR = 500,
    NOT_FOUND = 404
}
```
Add the resource to the `template.yml` file below our other function:
```yaml
...
GetPoll:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: ./dist/getpoll/getpoll.zip
      Handler: index.handler
      Runtime: nodejs6.10
      Policies: AmazonDynamoDBFullAccess
      Events:
        CreatePoll:
          Type: Api
          Properties:
            Path: /polls/{id}
            Method: GET
      Environment:
        Variables:
          TABLE_NAME: !Ref Polls
```

Generate our mock payload, referencing the id from the last section:
```bash
$ sam local generate-event api --method GET --path /polls/<id-of-poll-created-in-last-section> -r /polls/{id} > ./mocks/get-poll.json
```
We need to make a small modification to the mock file, as the `generate-event` command puts the entire route in a `proxy` path parameter instead of passing on the path params directly.  In a real API Gateway configuration, you can choose between these behaviors, which allows you to have a single function that can deal with wildcard routes, but remember that `sam-local` is an early beta and thus still has a few issues and missing features.  Open the `mocks/get-poll.json` file and find the `pathParameters` object.  Replace the `proxy` key with `id`, and delete the first part of the path in the value so that only the id is present:
```json
...
"pathParameters": {
    "proxy": "/polls/c2a7ca91-a598-4941-8f2e-5dda85482cb9"
  },
...
```
```json
...
"pathParameters": {
    "id": "c2a7ca91-a598-4941-8f2e-5dda85482cb9"
  },
...
```
Now we can test the new function. Don't forget to run `webpack` again first!
```bash
$ AWS_REGION="us-west-2" TABLE_NAME="poll-testdb" sam local invoke GetPoll -e mocks/get-poll.json
```
You should see some output with a json representation of the poll we created earlier.

===

Let's move on to the List function, which will return all of the polls in the database.