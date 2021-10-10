---
layout: post
title:  "Serverless Integration, Zero Code"
tags: ["aws", "serverless", "zero-code"]
---

> The words of Farah Campbell and Ben Kehoe on "The Serverless Mindset" has inspired me to level up my thinking and approach to Serverless integration. 
>
> So let me share my journey to zero code integration with you.

For my first foray into zero code integration, I decided to keep things simple and play with the classic integration pattern of sending data via an API into an SQS queue for downstream processing.  Most serverless websites will have you build an architecture like this, an API gateway triggering a Lambda that pushes to the SQS queue for downstream processing.

![Simple Classic Lambda Integration Architecture (with code)](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/upguplizspa6ubpucpxi.jpg)

This pattern is easy to build, deploy and get working with any of the main Serverless frameworks; my weapon of choice is the Serverless Framework from Serverless Inc.  The Serverless Framework focuses on deploying Lambda functions and makes this task quick and easy. For example, to build the Lambda processor pushing to SQS looks something like this.

```yaml
service: api-lambda-sqs

frameworkVersion: '2'

provider:
  name: aws
  runtime: python3.8
  lambdaHashingVersion: 20201221

functions:
  apiPushSQS:
    handler: handler.api_push_sqs
    events:
      - http:
          method: POST
          path: /
    environment:
      THE_QUEUE: !Ref downstreamSQS

resources:
  Resources:

    downstreamSQS:
      Type: AWS::SQS::Queue
```

Building out the pattern is simple, and the framework focuses on Lambda functions and their events which makes building and deploying quick and easy.  I managed to do this in minutes and felt productive; the instant win had my endorphins flowing, and I was conquering it!  

**But is this the ideal serverless solution?**

I am sure Ben Kehoe would say, "If you came along to my talk today to hear about Lambda, you are in the wrong place", and these words are echoing in my head right now as I write this and think how I can do this without code. 

We all know the AWS API Gateway supports direct, native integrations with many services, and I have designed many systems around these patterns. Still, I have never actually built one using the Serverless framework; until now.  

To achieve direct integrations from API gateway to other Services involves understanding how to configure the Integration request, and there are choices as to how we can go about this.  I started by configuring the API via the console and followed one of the few blog articles I could find on this topic to set it up, which you can look at [here](https://medium.com/@pranaysankpal/aws-api-gateway-proxy-for-sqs-simple-queue-service-5b08fe18ce50).  

This article allowed me to see the steps I needed to complete:

1. Create the SQS Queue
2. Create the IAM Policy allowing API Gateway to use the sqs:SendMessage action on my SQS Queue.
3. Create the role to attach the policy to and enable API gateway to assume the role.
4. Create the API and define a POST method that we can use to map the incoming request through to the SQS queue.
5. Integrate the API resource with our SQS Queue and create an integration mapping template to transform the incoming payload for our forwarding request to the SQS service.
6. Deploy and Test the API

Continue with me on my journey to complete these steps and end up with the following architecture, which uses zero-code.

![Zero-Code Serverless.yml](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/jbdv7d8fs489cyb533us.jpg)

To achieve a zero-code solution, we need to learn about the native AWS language of Velocity Templates and understand how data flows through the API Gateway service.  As Ben and Farrah would say - writing code is easy, but being genuinely serverless takes work, and it's not always the work we want to do or is fun to do, but we need to challenge ourselves to leverage the cloud services and write less code!

## How the API Gateway Data flow works

A quick high-level look at an API gateway request for a v1 REST API looks like this.

![AWS API Gateway Data Flow](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/401fkrejoihzh0dibyvv.png)

Each client request goes through the following stages for a simple un-authenticated request:

1. Method Request 
2. Integration Request - where transformations occur for the integration request to SQS
3. Backend Service (Simple Queue Service in this case)
4. Integration Response - where transformations occur for the integration response from SQS
5. Method Response - Mapping of integration Response to Method response for the client.

Now that we know the steps for the API request through the gateway, we can start setting this up using the Serverless framework because that's my current go-to for deploying serverless solutions.

## Create the SQS Queue

Creating the SQS queue is the easiest part of this solution. First, add the following to the resources section of the serverless YAML file.  I prefer not to provide the QueueName when setting up SQS; this allows the serverless framework to name the resources using the usual naming standard of serviceName-stage-resourceName-randomString.

```yaml
    sqsQueue:
      Type: AWS::SQS::Queue
```

## Create the IAM Policy and Role

To create the IAM Role and Policy, we cannot use the standard IAM methods for the Serverless Framework since these focus on setting up IAM Roles and Policies for functions in your solution.  This time there are no functions or Lambdas at all!
Add the role and policy details into the Serverless YAML file in the resources section.

![IAM Role and Policy](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/zc45qsy3wgn8pm7qweqh.png)

**Lines 23 - 34** enable API gateway to assume the role 

**Lines 35 - 44** define the actions allowed by the policy

**Line 44** uses Cloudformation intrinsic function !GetAtt to get the Arn of the SQS Queue we create in the stack.

## Create the Rest API

To create the Rest API, we need to create an AWS::Apigateway::Rest resource.  Usually, the Serverless Framework would make this for you under the covers as it creates functions triggered by HTTP events.  In the configuration for the RestApi, I have used `${self:custom.resourcePrefix}` for the "Name", which is a variable I set up to use for the naming of Services throughout my Serverless YAML.

![Create AWS RestApi](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/6pcsliy8zt4bmbo8229v.png)

## Create the API Resources and Integrate to our SQS

We create the  API method using the "AWS::ApiGateway::Method" resource, which needs to look like this.

![AWS API Method](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/mmoe5x1iwlng85mvowst.png)

Creating the API  method and integration is where all the work is in building out our zero-code solution and will break this down into the following steps for clarity:

1. Create the API Method and Responses (lines 60 - 70)
2. Create the integration - the easy bits (lines 71 - 79)
3. Create the Integration Request (lines 80 - 84)
4. Create the Integration Response (lines 85 - 99)

### Create the API Method

Creating the Method resource is the easy bit.  We set up the API Method on line 60 and Method responses on lines 64 - 69.  The HTTP status codes returned by the Integration response must be configured in the Method responses so that the API gateway will not throw an error and return an "HTTP 500 internal server error". 

On **line 61**, we use "!GetAtt apiGw.RootResourceId" to obtain the actual Id for the RestApi.

On **line 62**, we use "!Ref apiGw" to retrieve the RestApiId for our REST API. 

We have to use these Cloudformation intrinsic functions here since we want the values after AWS has created them during the stack deployment.

### Create the integration - the easy bits

I will break this down by the lines to make it easier to follow and explain each step.

**Line 71** defines the HTTP Method API gateway will use when calling the Integration endpoint; in this case, we want to use POST since we are sending data to SQS.

**Line 72** defines the type of integration; in this case, we are doing a direct service integration, so will use "AWS".  Using AWS means using an internal integration that allows us to transform the payload rather than "AWS_PROXY", which will proxy the request directly to the integration service leaving the body untouched.  

**Line 73** defines the IAM Role the API Gateway will assume while performing the integration.

**Line 74** defines the URI of the action we want to invoke.  The Uri action we need to invoke will look like "arn:aws:API gateway:us-east-1:sqs:path/1234567890/the_queue_name", so we are using the Cloudformation intrinsic function "Join" to create this for us.

Once we have these attributes configured, we have defined the core details on the service we are integrating, the "How".

### Create The Integration Request

**Line 80** defines the PassthroughBehaviour - I like to use "NEVER" so that if we receive an unexpected request "Content-Type", then API gateway will respond with an `HTTP 415 error - Unsupported Media Type`, which I think is desirable.

**Line 81 - 82** defines the request parameters we send to SQS; we will map our request into `x-www-form-urlencoded` values when sending our POST to SQS.

**Lines 83 - 84** define how we will transform the incoming body and send it to SQS; this is a simple VTL template setting the `Action=SendMessage` and `MessageBody=$input.JSON($)`, which is the JSON body of the request to be sent to the SQS queue. 

### Create the Integration Response

So far, we have created the API, formed the integration request and sent it to SQS; now, we have to deal with the SQS service response and map a response back to our caller using the "IntegrationResponses" attribute.  The value expected here is a MAP of HTTP status codes and VTL template snippets to transform the HTTP response we get from SQS.  In this demo, I have mapped the following response codes - 200, 404 to responses that will look something like the following:

**HTTP 200 Response**

We use a VTL template to transform the SQS response into a payload that we want to return rather than the entire integration payload.

```javascript
{  
	"request_id": "ea87ccb1-92d3-51ff-b9f0-956ad19844d7",  
	"message_id": "173689a1-6b40-492e-9e6c-338c13b70f16", 
	"accepted": "ok" 
}
```


**HTTP 404 Response**

```javascript
{
	"error_code": 404,
	"error_message": "Unable to complete Request"
}
```

## Deploy and Test the API

After creating the SQS Queue and a new REST API, we need to deploy our API to a stage so our clients can call the API.
Lines 101 - 108 define the Stage Deployment of our API and will make the API available.
Important Note: This resource will only work for the first deployment; any change to your REST API will not deploy as part of a stack update.  I searched far and wide for a solution to this behaviour and landed on the need to run a post-stack update API deployment using the console or AWS CLI.

```javascript
$ aws apigateway create-deployment --region <region> \
    --rest-api-id <api-id> \
    --stage-name <stage-name>
```

## The Wrap-Up

Writing a Zero-Code Serverless integration has been an evolving journey, and I have shared the details of where I ended up in this article.  I have learned a lot about AWS along the way and encourage you to explore and try code-less integrations in your solutions as a way of levelling up your journey to serverless nirvana.  What I have shared today is not production-ready - I am still exploring these patterns to make them more resilient and reliable for live use.

Check out the real working Serverless Solution (No code included) on my GitHub repository [here](