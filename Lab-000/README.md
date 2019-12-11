# TERRA 10 Fargate Labs

## Lab 001 - The first Fargate Task

## ECR - Elastic Container Registry
This is our repository for Docker Images. 
You can find it under ECS  Amazon ECR  Repositories. Or type in ECR under Services. This directs you directly to the repository.

Under Reposities you’ll find the repository colorteller and when opening this you will only find 1 Image with tag: latest. For this course it is good enough to have only the latest tag. However I wouldn’t recommend to use latest. Except maybe for snapshots.
Also keep in mind at any time no tag is given when loading an image ECS will pick latest.


## Task Definitions
With a task we define what we want to run and how to configure it.
Go back to Amazon ECS  Task Definitions. Or type in ECS under Services.

We are now going to create a task Manually.
Press Create new Task Definition
Select Fargate as our target. And press Next Step

!! Do note in all cases where you see Maarten rename this to your own name!!
It is just a name and not linked to your account.

Task Definition Name: Fargate<user>
Task Role:  ecsTaskExecutionRole
Task execution role:  ecsTaskExecutionRole

Task memory: 0.5
Task CPU: 0.25

Then we will find a blue button:  “Add container”.
This is to configure the container.

ContainerName: <user>
Image:  779717477382.dkr.ecr.eu-west-1.amazonaws.com/helloflogo
PortMapping – ContainerPort: 8080

The last thing we set are container environment settings:

``` 
postBeer:
  handler: lib/postBeerSimple.handler
  description: POST our new beers in DynamoDB
  events:
    - http:
        method: post
        path: beer
```

We can also delete the Hello example function because we won't be needing it anymore and compile & deploy:
``` 
tsc && sls deploy
```

## API endpoint & REST call
We will need the API endpoint from the AWS stage which is generated, so
* Check the AWS API Gateway: https://eu-central-1.console.aws.amazon.com/apigateway/home#/apis
* Select: dev-t10*-serverless -> Stages -> Dev
* There should be an invoke URL like: https://***********.execute-api.eu-central-1.amazonaws.com/dev
* Use a tool like Postman and fire a POST request to the url + /beer

Request:
``` 
{
  "beer_name": "Heineken",
  "beer_date": "2019-01-31"
}
```
If all goes "well" you should receive a 500 with response:
``` 
{
    "message": "User: arn:aws:sts::*:assumed-role/t10*-serverless-dev-eu-central-1-lambdaRole/t10a-serverless-dev-postBeer is not authorized to perform: dynamodb:UpdateItem on resource: arn:aws:dynamodb:eu-central-1:*:table/t10*-serverless"
}
```

## AWS Lambda roles
So now we have it. We can execute our first POST Lambda but on runtime it assumes a role which does not have permissions to dynamodb:UpdateItem on our table. So now what?

Well, add the role and permissions of course!

Add the following lines below the _provider_ segment in your serverless.yml:
``` 
  iamRoleStatements:
    - Effect: Allow
      Action:
        - dynamodb:UpdateItem
      Resource:
        - arn:aws:dynamodb:${self:provider.region}:*:table/${self:service.name}
```

The Serverless framework will help you to create an AWS IAM Role for the Lambda function and add IAM Policy permissions to insert an item in your database. This would be a very labour intensive step when done manually, but the Serverless framework does it all for you.

So deploy again (we don't need the compile here, but it's better to get used to doing it anyway):
``` 
tsc && sls deploy
```

## Try the API call again
Request:
``` 
{
  "beer_name": "Heineken",
  "beer_date": "2019-01-31"
}
```
If all goes well you should receive a 201 with a response:
``` 
{
    "id": "xxxxxxxxxxxxx"
}
```

## Summary
So let's check:
* Items in DynamoDB table: https://eu-central-1.console.aws.amazon.com/dynamodb
* Logging in CloudWatch: https://eu-central-1.console.aws.amazon.com/cloudwatch
* POST API in API Gateway: https://eu-central-1.console.aws.amazon.com/apigateway

## The second POST function (you can skip this but it shows some advanced features)
Check the reference/postBeerAdvanced.ts file and see how we can handle JSON objects which might contain optional or unknown elements which we always want to store without doing massive amounts of checks in our code.