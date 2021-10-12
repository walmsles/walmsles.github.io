---
layout: post
title: "Correlating Transactions with Zero Code"
tags: ["aws", "serverless", "zerocode"]
---
> I take pride in writing detailed, technical articles filled with solid code examples and links to Github so you can review and learn from my serverless journey. Today, I think this will be my shortest ever article, and it lacks all these things since starting on this Zero Code adventure.

  

There are a lot of articles out there on correlating transactions across distributed systems and plenty of technical frameworks and solutions for providing transaction traceability. However, correlating transactions as the data passes through our services is essential to observe our system behaviour. So, as developers, we reach for what we know best our development tools, software frameworks and technical solutions. 

Adding Correlation Ids, tracing codes and segment identifiers takes effort and time.

Usually, we place these identifiers in HTTP Headers and other unique Attribute locations for Cloud-Native services so we can hide these away and provide them as transparent travellers across our distributed systems. In an AWS Managed service sense, this means we need to know how to pack and unpack these identifiers at each step through our serverless systems - across SQS boundaries, through EventBridge events and via SNS topics. These cloud-native services each have a different mechanism to save, transport, and unpack these keys to observability.

  

**But what if there was a different way without all this complexity and code?**

  

I want to share with you my thinking on this and what I have arrived at to resolve this technical problem in a way that transcends the AWS service complexities I just mentioned. Nowadays, I believe in using the language of integration, and for tracing transactions through distributed services, I create an internal integration language encapsulating these observability identifiers.

Here is an example of a message structure from a system I am working on right now:

  
```json
{
    "correlation_id": "b58e7f98-2b58-11ec-8d3d-0242ac130003",
    "message_id": "bde284dc-2b58-11ec-8d3d-0242ac130003",
    "trace_id": "3965a72e-2b59-11ec-8d3d-0242ac130003",
    "data":
    {
        "id": "7e6c7638-218c-4947-8d84-a20a9cf68acd",
        "firstname": "michael",
        "lastname": "walmsley",
        "email": "michael@myemail.com"
    }
}
```

The **data** represents the original payload received from the initial caller of the service, usually via an API endpoint.   The API entry point is responsible for validating the data to ensure it is correct before transforming it into the internal message format containing the tracing identifiers (correlation_id, message_id, tracing_id). Using a standard messaging structure like this means we no longer worry about changing how we interact with each of the Managed Services we are using since our correlation data is now directly embedded within our message data. All of our messages now have correlation identifiers that pass through data services like SQS, Eventbridge, SNS, Kafka, Kinesis, etc., without the need for any custom code to embed meta-data or other custom markers for correlating our transactions. 

This simple approach of defining an internal service messaging format allows your transaction data and correlation data to always be visible within every component of your distributed Serverless Architecture. Furthermore, we achieve Transaction observability within our Serverless data processing systems using nothing more than our already prepared Cloudwatch Logging utilities and Zero additional code!
