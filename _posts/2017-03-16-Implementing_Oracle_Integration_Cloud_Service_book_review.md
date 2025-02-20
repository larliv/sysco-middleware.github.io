---
layout: post
title: Noobs notes (or another "expert" opinion) - “Implementing Oracle Integration Cloud Service” book review
categories: oracle cloud paas
tags: [Oracle, Cloud, Integration, Review, Noobs notes]
author: sergei-ko
---

If words "Cloud" and "Integration" are something you are interested in and you are in search of some products/services in this area, then this book review is for YOU!

This is a book review on the book “Implementing Oracle Integration Cloud Service” written by Robert van Mölken and Phil Wilkins – on PACKT. For more information visit this [link][1].

![](/images/2017-03-16-Implementing_Oracle_Integration_Cloud_Service_book_review/BookCover.jpg)

Book Details:

ISBN 139781786460721

Paperback 506 pages

Book contains the following chapters:

* *Introducing the Concepts and Terminology*
* *Integrating Our First Two Applications*
* *Distribute Messages Using the Pub-Sub Model*
* *Integrations between SaaS Applications*
* *Going Social with Twitter and Google*
* *Creating Complex Transformations*
* *Routing and Filtering*
* *Publish and Subscribe with External Applications*
* *Managed File Transfer with Scheduling*
* *Advanced Orchestration with Branching and Asynchronous Flows*
* *Calling an On-Premises API*
* *Are My Integrations Running Fine, and What If They Are Not?*
* *Where Can I Go from Here?*

Let’s look at each of these chapters in details:

**Introducing the Concepts and Terminology**

This chapter does not describe everything you need to know when it comes to basic knowledge about the integration, but just enough for the reader to understand how communication between applications happens in Oracle Integration Cloud Service. Variety of adapters, On-Premise and Cloud communication with the usage of different agents, several integration types, transformation and lookups.

**Integrating Our First Two Applications**

This section concentrates on example of how you can integrate different services, in our case SOAP inbound and REST outbound hosted on Apiary, with Basic Map Data style/pattern. Detailed description of the process on how to create, monitor and test (with SoapUI) the integration including some security issues overview. Brief introduction to Apiary and API Blueprint was very appropriate.

**Distribute Messages Using the Pub-Sub Model**

This chapter dedicated to the description of how to perform publish and subscribe pair of integrations, and then extend it with additional subscriber integrations. This time Mockable.io was used to mock web-service. Detailed instructions were given on how to create and test integration. Brief introduction to Mockable.io was very appropriate.

**Integrations between SaaS Applications**

In this section authors showed how to integrate two SaaS applications on the example of integration between Salesforce and Twilio in the ICS. As a prerequisite some preparation were made in Salesforce (workflow rule and outbound message) and in Twilio (set up of the SMS messaging service and required phone numbers). It also explains the ease of use of some of the out-of-the-box cloud adapters (in this case Salesforce adapter). Some testing and troubleshooting were also performed as an example.

**Going Social with Twitter and Google**

To continue showing what ICS is capable of, authors describing example of how social media could be integrated also. First example explains how to integrate with Twitter for posting update as a message and usage of out-of-the-box Twitter adapter in ICS. Second example describes integration with GoogleMail for sending report by email through out-of-the-box Google Mail adapter. As in previous chapters some testing and troubleshooting were performed.

**Creating Complex Transformations**

This chapter contains example of how it is possible to enrich message through the usage of additional variable that holds the enrichment data. It also states that it is possible to do enrichment from both external and internal services. Another feature which is shown here is lookup mechanism and how to see functions documentation.

**Routing and Filtering**

Just as headline states this section is about filtering and routing. Simple and extended versions of filtering were shown. SOAP to SOAP and REST to SOAP integrations were performed (with Mockable.io in the back). Message content based routing functionality was also applied (with two different endpoints). There was also a deeper dive into the REST endpoint properties.

**Publish and Subscribe with External Applications**

According to the book "ICS publish and subscribe mechanism only works within ICS and cannot be accessed directly from external applications". So, in order to show how to integrate with external/third-party solutions, authors demonstrate integration using Oracle Messaging Cloud Service (enterprise messaging solution). It is shown how Oracle Messaging Cloud Service (OMCS) makes this integration possible. As an example, sample application used that allowed to send and consume messages using the JMS standard implemented by OMCS.

**Managed File Transfer with Scheduling**

Useful intro on difference between File and FTP connectors starts the chapter. After that setup of the FTP and then FTP to FTP integration takes place. Manual trigger and Scheduler (automatic trigger) functionality also described in details. There is also a practical example of the file encryption and decryption for FTP adapter. This chapter also touches orchestration (particularly with FTP) topic, but very briefly.

**Advanced Orchestration with Branching and Asynchronous Flows**

This chapter explains Orchestration pattern and shows it on the complex step-by-step example including branching and asynchronous flows. Small intro on what is Trello and how to setup it. Extensive description of orchestration integration, including re-use of created earlier integrations, and its testing occupies the whole chapter.

**Calling an On-Premises API**

Chapter is dedicated to cloud - on premise integration and, in particular, on-premises agent: what is it, types, when to use, how to use and how to deploy. The differences between connection and execution agents were explained. Rest of the chapter was dedicated to go through integration itself.

**Are My Integrations Running Fine, and What If They Are Not?**

In order to understand on how to monitor integrations "health" (including all the components around them like connector, agents etc.) and troubleshoot issues, authors introduce in this chapter the variety of different monitoring information sources present in ICS. Besides monitoring there are also several administration capabilities described here like logging setup, emailing reposts etc., which are also contributing to issues resolution. In the final section authors describe on how it is possible to incorporate all these into an enterprise-wide monitoring platform.

**Where Can I Go from Here?**

In the last chapter of this book authors concentrate on several things: import/export of integrations, connections etc., different scenarios of usage and different approaches (individual and package import/export); ICS API, with its control and monitoring capabilities including sufficient introduction of cURL; Cloud Adapters Software Development Kit (sometimes shortened to Adapter SDK), which enables possibility to create custom adapters that can be used by ICS. In the final section of this chapter further reading on different aspects of integration (patterns, standards etc.) is presented.


**Summary:**

Oracle Integration Cloud Service is a part of Oracles effort to reach for their competitors in the cloud. This book is a detailed look at what this cloud service is capable of. It describes quite an extensive amount of functionality and many different tools. Authors put in extra effort to introduce us to several tools/services/languages outside of the integration cloud scope (Apiary, Mockable.io, API Blueprint, Salesforce, Twilio etc.). It is possible to say that they have done some good job to show that ICS can be integrated with many different technologies in a simple way without very complex development.
Book consists mainly from the different usage examples of the Integration Cloud. "Down to earth", example-based structure of the book makes it easy to read and understand some particular functions and usage scenarios.

At one of the Oracle hands-on trainings I was able to go through some of the similar labs which are present in this book and can say that this books does a decent job on helping readers to visualize by supplying generous amount of pictures and diagrams.

It is safe to say that this book is oriented on developers and sales/pre-sales specialists who want to learn about particular Oracle service and already have some general knowledge and understanding of integration principles and concepts. It is not a book for beginners or those who want to know more about cloud in general or about integration.





[1]: https://www.packtpub.com/virtualization-and-cloud/implementing-oracle-integration-cloud-service
