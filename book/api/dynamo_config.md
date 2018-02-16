# Persistence Configuration
To store our data, we'll use AWS's answer to document or object based NoSQL databases, DynamoDB.  Luckily for us, SAM has us covered here with the `AWS::Serverless::SimpleTable` CloudFormation resource, which is a simplified syntax for creating tables with a single-attribute primary key.

Open `template.yml` and add the following to the top of the `Resources` section:
```yaml
Resources:
  Polls:                   
    Type: AWS::Serverless::SimpleTable
    Properties:
    PrimaryKey:
      Name: id
      Type: String
    ProvisionedThroughput:
      ReadCapacityUnits: 5
      WriteCapacityUnits: 5
```
Here we are directing CloudFormation to provision a DynamoDB table called `Polls` with a string primary key called `id`.  We're also setting provisioned throughput for the table, which is an important consideration but beyond the scope of this tutorial (I'll do a DynamoDB deep dive at some point in the future).  5 read and 5 write units should be sufficient for our purposes.  Note that we are not defining any kind of schema or data structure, this being a NoSQL data store.
