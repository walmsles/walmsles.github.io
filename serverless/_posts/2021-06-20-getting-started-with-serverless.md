---
layout: post
title:  "Getting Started with Serverless"
tags: ["aws", "serverless", "lambda"]
---
I started learning about serverless development and AWS lambda in 2016 while working with startup companies building web APIs and single-page apps. These companies were Digital Agencies building out products to disrupt their industries and since they knew how to manage websites, server-based web hosting was the usual deployment environment. At this time I was heavily invested in docker containers and container orchestration and started introducing Docker workflows to these companies as a means of shrinking deployment footprints and standardising environment control from development to production.

It was this interest in perfecting and minimising deployment footprints and infrastructure management that made AWS lambda stand out for me as a technology choice for building out web APIs. Since learning about AWS lambda and its capabilities I have not returned to server or container approaches to building software, instead, I have been more focused on cloud-native services and AWS lambda. It has been over 4 years since I last deployed a server on AWS or any other provider and I feel cured of the need for servers forever.

![Cloud Network](/assets/serverless/network-4851119_1280.jpg)

My journey to building full-time with cloud-native services is a winding road of failings and learnings as I started to understand how to control all the puzzle pieces that AWS provides and how to combine these into a cohesive system that was resilient, robust, and highly scalable. In the remainder of this article, I want to highlight what I feel are the main considerations when trying to build out serverless solutions on AWS, a snapshot of my experience to date, and the main considerations for getting started.

## Keeping it Simple
One of the key evolutions of moving to Serverless is that of writing only the code that adds direct business value to your solution. Keeping your code small, concise and simple is important for several reasons:
A smaller code footprint means faster startup time when lambda is starting for the first time (minimising cold starts).
Less code means faster development iterations, so quicker delivery cycles
Less code means simpler code and also means testing is simpler or perhaps non-existent since there is less complexity in your code. Unit testing is definitely required for more complex or shared code to ensure correct operation but striving for 100% coverage is not as important.

>Keep It Simple Stupid (KISS)

## Know Your Services
Developing solutions with AWS lambda you really have to understand how the managed services you are using will trigger lambda, how they need to be configured, how they could fail, and how you notify the service that your lambda has failed to process the event or has been successful. I feel this is the main change in thinking and attitude, serverless developers need to understand all the components they are using in-depth and to favour writing configuration over code. AWS provides managed services with a guaranteed SLA but sometimes these services just misbehave or behave in unusual ways so we need to understand how the services work and design with failures in mind.

>Know your Services, understand how they can fail

## Understanding Your Service Limits
This has to be one of my key learnings for designing and building cloud-native services. When building any solution we have to understand what the underlying "infrastructure" or managed service quotas and limits are. Without understanding the key limitations which AWS imposes to guarantee throughput and resilience of their managed services you really are starting to guess at how well your solution will scale and behave. Every AWS service has a defined set of limits and quotas which you need to understand so that your solution will work in every situation.

>Know all your limits to guarantee success

## Know Your Volumes
In my experience not understanding your volumes is a recipe for disaster, this is not new for serverless development it applies to container or server-based systems as well. I would have to say the biggest mistake any developer can make is not understanding the transaction volumes that will be processed by your system. AWS Lambda will scale and this is fine but you have to be sure that any downstream integrations or services you make use of are also capable of scaling in exactly the same way. I would say this is the most important point for designing and building scalable distributed systems that are reliable and resilient.

>Know all your volumes, take the time to really know how your service will be used.

## Understand Scalability Boundaries
Following on from "Know Your Volumes" is my next key learning, understand how different sections of your solution scale. Not all managed services scale at the same rate or sometimes we have systems we interact with that are not highly scalable like AWS lambda. In my experience to date, I would say this is the number one mistake made by developers who are starting out. They underestimate the scaling nature of lambda and when it starts to scale out of control every component in your AWS account is at risk, this also links directly back to knowing your volumes. Not considering how scalability boundaries clash is definitely the most common cause of failure when starting out with serverless development.
Watch for Scalability boundary clashes which will always be high scale pushing into lower scale or even connection-oriented services

## Visibility of Your Services
Operational logging is the most critical function in any distributed service. Without clean, concise logging you will not be able to understand what your system is doing. This is not unique to serverless development but in my experience when developers are put under pressure to deliver, logging is the last thing that is considered. Without clear and useful logging in place, it becomes more difficult to see what is happening within your services and almost impossible to know what your system is doing.

>Make sure you can see your transactions and how they flow through your system from end to end, don't underestimate good logging

If you are just getting started with AWS lambda and cloud-native development take a moment to consider these themes and apply them as you start to learn and build on AWS with lambda and cloud-native services.

