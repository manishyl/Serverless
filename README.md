# Serverless
APIGW->Lambda-> DDB


![image](https://github.com/user-attachments/assets/2391d93c-5f76-4896-bafe-dffdf452053c)


1. Lambda Role - to access DynamoDB abd CloudWatch logs
   {
"Version": "2012-10-17",
"Statement": [
{
  "Sid": "Stmt1428341300017",
  "Action": [
    "dynamodb:DeleteItem",
    "dynamodb:GetItem",
    "dynamodb:PutItem",
    "dynamodb:Query",
    "dynamodb:Scan",
    "dynamodb:UpdateItem"
  ],
  "Effect": "Allow",
  "Resource": "*"
},
{
  "Sid": "",
  "Resource": "*",
  "Action": [
    "logs:CreateLogGroup",
    "logs:CreateLogStream",
    "logs:PutLogEvents"
  ],
  "Effect": "Allow"
}
]
}

2. Lambda Function
```
from __future__ import print_function
import boto3
import json
print('Loading function')
def lambda_handler(event, context):
    '''Provide an event that contains the following keys:

      - operation: one of the operations in the operations dict below
      - tableName: required for operations that interact with DynamoDB
      - payload: a parameter to pass to the operation being performed
    '''
    #print("Received event: " + json.dumps(event, indent=2))

    operation = event['operation']

    if 'tableName' in event:
        dynamo = boto3.resource('dynamodb').Table(event['tableName'])

    operations = {
        'create': lambda x: dynamo.put_item(**x),
        'read': lambda x: dynamo.get_item(**x),
        'update': lambda x: dynamo.update_item(**x),
        'delete': lambda x: dynamo.delete_item(**x),
        'list': lambda x: dynamo.scan(**x),
        'echo': lambda x: x,
        'ping': lambda x: 'pong'
    }

    if operation in operations:
        return operations[operation](event.get('payload'))
    else:
        raise ValueError('Unrecognized operation "{}"'.format(operation))
```

Test:
{
    "operation": "echo",
    "payload": {
        "somekey1": "somevalue1",
        "somekey2": "somevalue2"
    }
}

3. DynamoDB Table
   Create a table with the following settings.
   Table name – lambda-apigateway
   Primary key – id (string)

4. API Gateway : Create API -> Resource -> Method(Post) and choose the Lambda Function created earlier.

5. Test :
   Using Postman : Us ethe POST method with Invoke URL from API GW Resource/Method
      {
    "operation": "create",
    "tableName": "lambda-apigateway",
    "payload": {
        "Item": {
            "id": "1234ABCD",
            "number": 5
        }
    }
}


Learnings: 
- Use API + RESOURCE URL instead of just the API URL else you will run into { message: "Missing Authentication Token"}
- DynamoDB updates every 6 hours so the items may not be immediately available to see on the console. Try doing a list to check the contents instead or run a scan on the table. 
