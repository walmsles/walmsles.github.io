---
layout: post
title:  "Making All Your APIs Idempotent"
tags: ["aws", "serverless", "aws lambda powertools", "lambda"]
---

> When you start working with Serverless on AWS, one of the first questions you should be asking yourself is — “is my system Idempotent”?

## What is Idempotency?

Idempotency is a straightforward but misunderstood concept. The dictionary defines Idempotency as “*Unchanged when multiplied by itself*”.

![img](https://cdn-images-1.medium.com/max/1600/1*e10oxJhKCYzrlFh1NWxqfQ.png)

In a micro-service context, this means any repeated action within the service will always have the same effect. The AWS Builders Library has a great article covering [Idempotency](https://aws.amazon.com/builders-library/making-retries-safe-with-idempotent-APIs/) and how AWS implements Idempotent measures within their distributed systems. This concept makes writing API client error handling simpler, and I highly recommend reading the AWS article!

Implementing Idempotency will ensure your API will always respond to clients in the same way for the same input — this last part is essential — ***for the same input.\*** Idempotent solutions require your micro-service to store the state of each response to an API call so you can return the same payload for any repeated call. It is up to you to define what makes an API call original so that you can store the Idempotent response for replay later. Once you have this, you need to decide how long the initial response should persist for repeated calls. The length of time you cache API responses depends on your overall solution, so careful consideration is needed during service design to get this correct for your situation.

## Why do I need this?

When designing a new cloud service, Idempotency should be the first tenet considered before building anything. Idempotency is needed because any external interaction over an API is likely to fail some of the time. The following diagrams show API scenarios and possible request-response scenarios for distributed API driven systems.

![img](https://cdn-images-1.medium.com/max/1600/1*cJESb5xr_4EJQjg000BTqA.jpeg)**Scenario 1: All Okay**

![img](https://cdn-images-1.medium.com/max/1600/1*TXMCG7vc3rYVS-P3GJXbJw.jpeg)**Scenario 2: Clear Error Response**

![img](https://cdn-images-1.medium.com/max/1600/1*uw-EGaoznpKe25eaQqKYcg.jpeg)**Scenario 3: Client request Times out**

![img](https://cdn-images-1.medium.com/max/1600/1*oiPW614dWRqFEbm-SVQLlA.jpeg)**Scenario 4: Server Response never Received**

Scenarios 1 and 2 represent the usual test cases for your API. These are the most straightforward test cases. Scenarios 3 and 4 are the tricky cases and are the main reasons we must consider implementing Idempotency. Both of these scenarios for the API client are identical in behaviour; they both appear as API timeouts to the client. Client time outs or network issues where clients never receive a response feels like an unusual occurrence. It won’t usually be an issue during development or testing — only when your system is in use over a more extended period and at high volume will these scenarios start to emerge, often that is after you have gone live. Without Idempotency in place, any client timeout or non-response (which does happen!) means a client will retry the API action. In distributed systems, without Idempotency, this could result in duplicated data on the target platform, which is undesirable in all cases. Many implementations will take the simple approach of returning a duplicate record error and feel this is handling their Idempotency. In a way, it does, but now every client making the API call needs additional error processing to recognise the timeout case followed by a duplicate error means the original interaction was successful. Implementing Idempotency in all your APIs is critical for success in your distributed systems.

## Implementing Idempotency

Creating an idempotent API is easy if you have access to the right tools! I am currently implementing APIs using AWS lambda and python as my runtime, making the [AWS Lambda Powertools for Python](https://awslabs.github.io/aws-lambda-powertools-python/latest/utilities/idempotency/) an effortless choice which implements a great utility that deals with Idempotency for you out of the box. In addition, the AWS Lambda Powertools for Python provides several useful tools and utilities to ensure you can easily follow the AWS Well-Architected Framework. Heitor Lessa from AWS contributed to the Serverless Lens version of the Well-Architected Framework and is the creator of the AWS Lambda Powertools for python, which provides simple tools and utilities to apply the tenets of the framework to all your Lambda projects.

Implementing Idempotency with the Lambda Powertools for Python is very straightforward; the community behind this open source project have thought about almost everything! The key to Idempotent APIs is defining what makes each API interaction with your service unique; it could be the entire body of your API or a specific data element provided by the caller. Once you have determined what makes each of your APIs Idempotent, it is time to introduce the Idempotent decorator from the Powertools library. The Idempotent utility in Powertools uses DynamoDB as the persistence layer. Therefore, if you do not have a DynamoDB table in your project already, you will need to add one to your Infrastructure as Code to make sure it exists.

![img](https://cdn-images-1.medium.com/max/2400/1*fBeNXijMzo2h-BsSnS1rfg.png)**example implementation of Idempotecy**

The above example implements Idempotency using the entire API body to provide uniqueness for my API. Let’s walk through the code in more detail:

**Line 4:** Imports the required classes from Powertools to achieve Idempotency

- **DynamoDBPersistenceLayer** — stores Idempotent data in our DynamoDB table.
- **IdempotencyConfig** — Provides configuration for the Idempotentcy decorator.
- **Idempotent** — this is the decorator providing the utility.

**Lines 6–7:** Creates the persistence layer for Idempotency; in this example, I am using a Lambda environment variable to store the name of the DynamoDB table to use.

**Line 9**: This line defines the configuration of uniqueness for the API call. In this example, I use the **body** attribute of the Lambda API Gateway event to determine a unique transaction. Using the entire Lambda Event is impossible since AWS will always include unique request-ids and other AWS specific data. Using the **body** attribute means the whole payload for my API defines uniqueness.

**Line 11:** Adds the **idempotent** decorator to your lambda handler passing in **persistence_layer** and **config**; this is where the Idempotency magic happens! If a duplicate API request is received and the response cache has not timed out, Powertools won’t call your Lambda handler. It will return the cached response from the first API call instead.

Like every utility in the AWS Lambda Powertools, implementation is quick, easy and painless. In my code example, I have achieved Idempotency for my serverless API with five lines of code. Every utility in the Powertools is equally effortless to implement, and I highly recommend adopting the AWS Lambda Powertools in every serverless project you create! I have shown an example today using python, but AWS Lambda Powertools also supports other runtimes — Java, C# (.NET) and Typescript. With these Open Source projects freely available and well supported in the community, there is no longer any excuse for not adopting AWS Well-Architected best practices for your Serverless projects!

I have only highlighted the Idempotency utility; there are so many more utilities available to implement. I recommend you review the complete documentation for [AWS Lambda Powertools for Python](https://awslabs.github.io/aws-lambda-powertools-python/latest/) and see how easy it is to implement a Serverless project with best practices baked in!