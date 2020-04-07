

**What are we doing?**

We're going to build a serverless backend and infrastructure for an React app using AWS CDK(Cloud Dev KIt)

**AWS CDK You can create cloud resources using a lang you know** 

 CDK is built on AWs CloudFormation and AWS service that lets you describe a stack using YAML or JSON

Will convert TS to JS into CloudFormation  

CDK abtracts a lot from us we we can get things done 

CDK Basics

bin dir file that creates our stack

`cdk init sameple_app --language-typescript`

will create stack file

`import @aws-cdk/core` is required

anything else you want import is for what you want to use in your app

`sqs.Queue` are construct in CDK

construct is an instance or SOMETHING that will be provsied in a AWS cloud, You can create your own

Every construct takes 3 arguments 
- scope -usually this
- id - an identifier
- props

```js
const queue = new sqs.Queue(this, "TestCdkQueue", {
            visibilityTimeout: cdk.Duration.seconds(300),
        })
```

CDk Synth - generates a cloudFormation Template 


**Question from Chat:so regarding this as scope it is going to be always a class that extends Stack?**

A: not always

Use AWS console to view stack 

We don't need queue or topic so we can deleted in this example

`cdk - diff` in the terminal tells you if you deploy it will deleted those files

**Question fomr the chat: Would I duplicate stacks to handle dev/prod environments?**

YES 

**Question from the chat:Why does it deleted queue and topic? IF it creates them automatically?**

So we can only deploy the things we need

**First Hello World Lambda Function**

aws lambda function - provide piece of code to AWS and stored waiting to be executed 

```js
import * as lambda from "@aws0cdk/aws-lambda"
```

```js
const helloLamba = new lamba.function(this, MyHelloWorld", {
	runtime: lambda.Runtime.NODS_12_X,
	code: lamba.Code.asser("lesson_01/lambda"),
	handler: "Hello.handler"
});
```

lambda can have different runtimes
You code path where file is located

AWS does config defaults

To test function you can config a test event

**Question from the chat:so to have different values for different environments you would use something like dotenv?**

Yes, absolutely


**With lambda functions you want it to fail ASAP so you won't be charged too much, that's why you don't set the timeout really high.**

15 mins is the max timeout it will be killed once over 15 mins

Make sure you check your region for CloudFormation 

**Attaching an API Gateway to our lambda function**

Lambda function are suppose to fire on an event trigger

**Amazon API gateway** - lets you call a lambda function from the internet 

Import a `@aws-cdk/aws-apigateway`  into .stack file

only handler is required we'll use `helloLambda`

```js
new apiGateway.LambdaRestApi(this, "Endpoint", {
	handler: helloLambda
});
```

When you deploy you get a URL that you can call the function on the web 

**Executing a lambda function locally**

**AWS SAM** lets us call lambda functions locally

Run `cdk synth --no-staging > template.yaml` to create a CloudFormation template and store it in a template.yaml file

copy the logical id then run `sam local invoke HelloLambda3D9C82D6 -e sample_events/hello.json`  to invoke the function

**Resource for AWS SAM**

https://egghead.io/playlists/learn-aws-serverless-application-model-aws-sam-framework-from-scratch-baf9


**Amazon S3** - Simple Storage Service

Start by @aws-cdk/aws-s3

then create new bucket
`const logoBucket = new S3.Bucket(this, "LogoBucket");`


To trigger event when img is uploaded `import * as @aws-cdk/aws-s3-notifications`

```js
logoBucket.addEventNotification(
	s3.EventType.OBJECT_CREATED,
	new s3Notifications.LambdaDestintion(helloLambda)
);
```


**Building custom AWS CDK constructs**

To make our app easier to maintain we create a new file in the lib directory called `todo-database.ts`

```js
import * as cdk from "@aws-cdk/core";

export class TodoDatabase extends cdk.Construct {
  constructor(scope: cdk.Construct, id: string, props?: cdk.StackProps) {
    super(scope, id);
  }
}
```

Import and create an instance of the construct in todo-app-stack.ts 

`import { TodoDatabase } from "./todo-database;`

**Resource for intro in DynamoDB**

https://egghead.io/playlists/intro-to-dynamodb-f35a

**Question from the chat:Is it possible to pass any params to a custom construct?**

Yes

Import dynamoDB 

and create a new table 

```js
import * as cdk from "@aws-cdk/core";

export class TodoDatabase extends cdk.Construct {
  constructor(scope: cdk.Construct, id: string, props?: cdk.StackProps) {
    super(scope, id);
  }
}
```

```js
`new dynamodb.Table(this, "TodoTable", {
            partitionKey: { name: "id", type: dynamodb.AttributeType.STRING }`
```


**Reading data from a DynamoDB table with a Lambda function**

**Resource to learn how to use Node to interact with dynamoDB**

https://egghead.io/playlists/dynamodb-the-node-js-documentclient-1396

to implement getAllTodos()
In todoHandler.ts

```js
const getAllTodos = async () => {
    const scanResult = await dynamo
        .scan({
            TableName: tableName
        })
        .promise();

    return scanResult;
};
```



**Question from the chat: are there any architectural patterns and good practices for CDK? like a recommended way to structure your constructs?**

https://github.com/cdk-patterns/serverless
CDK is fairly new but you can use this resources

**Debugging Lambda + DynamoDB issues and managing permissions in the Cloud**

AWS follows the Principle of Least Privilege. The idea is that every user, resource or process in AWS gets the minimal amount of permissions that are required to get the job done.

We can grant  Lambda functions access to a DynamoDB table

Luckily, with AWS CDK this is really simple. Use the grantReadWriteData function from the `todoTable` in order to allow our `todoHandler` to read/write data from this table.

```js
todos.Table.grantReadWrtiteData(this.handler);
```


**Connecting frontend app to an AWS CDK infrastructure**

Create.ENV file and copy URL we used earlier to the endpoint in the ENV file


**Question fomr the chatis there a way to output the stack outputs when we execute cdk deploy from the SDK? 
 like writing it to an external filles?**

Yes and we will do it in the workshop 


Add Access-Control-Allow-Origin: *" and Access-Control-Allow-Methods:
OPTIONS,GET,POST,DELETE headers to our createResponse function


Make sure that OPTIONS request returns an OK response. To do that - add

```js
if (httpMethod === "OPTIONS") {
    return createResponse("ok");
}
```

Build app by `running npm run build` in frontend dir

In `todo-app-stack.ts` create a new public bucket for our website websiteIndexDocument


create a new BucketDeployment 

```js
new s3Deployment.BucketDeployment(this, "DeployWebsite", {
            destinationBucket: websiteBucket,
            sources: [s3Deployment.Source.asset("../frontend/build")]
        });
```

Add 

```js
new cdk.CfnOutput(this, "WebsiteUrl", {
	value: websiteBucket.bucketWebsiteURL
}); 
```
so we'll have an easy access to our app once the stack is deployed


**ADD CDN for speed**


Delete website bucket, bucket deployment and `CfnOutput` from `todo-app-stack.ts`



 `import { SPADeploy } from "cdk-spa-deploy";` to `todo-app-stack.ts`



```js
new SPADeploy(this, "WebsiteDeployment").createBasicSite({
            indexDoc: "index.html",
            websiteFolder: "../frontend/build"
        });
```


**Question from the chat: your experience has been any situation in which you had to rely in lower level solutions because you could not achieve something using CDK?**

Yes sometimes you have to do things on your own
