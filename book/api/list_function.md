# Authoring the List Function
We need a way to get a complete list of the polls that have been created.  To do this, we'll utilize the *scan* operation provided by DynamoDB.  It's imporant to note that scanning a table is an expensive operation, and should be avoided if the alternative *query* interface will suffice.  Essentially a scan involves retrieving the entire table and then optionally applying filters, while a *query* is more like a SQL *select* in that the database itself is doing the heavy lifting.  If your query only invoves the hash key and/or sort key, query is the way to go.  In this case, though, we actually want to retrieve the entire dataset so it's appropriate.  

Another important thing to keep in mind is that DynamoDB limits the size of responses to around 1MB, and it handles this through a pagination system.  If your table contains more than a handful of records, you **must** write your code to utilize the pagination scheme.  For our purposes, we're going to overlook this.

Again we'll begin by extending the `DynamoRepository` class to include this ability:
```typescript
public getAll() : Promise<ScanOutput> {
        let params = {
            TableName: this.tableName
        } as ScanInput;

        return new Promise<ScanOutput>((resolve, reject) => {
            this.dynamoClient.scan(params, (err, data) => {
                if (err) reject(err);
                else resolve(data);
            });
        });
    }
```
And create our function file, `src/list-polls.function.ts` with the following:
```typescript
import { Handler, APIGatewayEvent, Context, Callback } from 'aws-lambda';
import { Poll } from './poll.model';
import { DynamoRepository } from './dynamo-repository';
import { ApiGatewayResponse, ResponseType } from './api-gateway-response';

export const handler: Handler = (event: APIGatewayEvent, context: Context, callback?: Callback) => {    
    new DynamoRepository<Poll>(process.env.TABLE_NAME).getAll().then((val) => {
        callback(null, new ApiGatewayResponse(ResponseType.OK, val.Items));
    }).catch((err) => {
        return callback(null, new ApiGatewayResponse(ResponseType.SERVER_ERROR, { status: false, error: err }));
    });
}
```
Update the `template.yml` to include the function:
```yaml
ListPolls:
    Type: AWS::Serverless::Function
    Properties:
        CodeUri: ./dist/listpolls/listpolls.zip
        Handler: index.handler
        Runtime: nodejs6.10
        Policies: AmazonDynamoDBFullAccess
        Events:
        CreatePoll:
            Type: Api
            Properties:
            Path: /polls
            Method: GET
        Environment:
            Variables:
                TABLE_NAME: !Ref Polls
```
Create the mock payload:
```bash
$ sam local generate-event api --method GET --path /polls -r /polls > ./mocks/list-polls.json
```
Run `webpack`, then test the function:
```bash
$ AWS_REGION="us-west-2" TABLE_NAME="poll-testdb" sam local invoke ListPolls -e mocks/list-polls.json
```
And you should see a response, again containing the same poll we created earlier.

---

Moving right along! Let's knock out the update function next.
