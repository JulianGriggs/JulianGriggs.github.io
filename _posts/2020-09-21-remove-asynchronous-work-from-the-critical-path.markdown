---
title: Remove Asynchronous Work from the Critical Path
layout: post
---
Our APIs often involve a mix of work that needs to performed before a success/fail response can be sent and some work that needs to happen, but doesn't need to happen immediately. Work that must happen along the request path we say is **synchronous**. Work that must happen *eventually* we call **asynchronous**. When writing high performant APIs, we should strive to remove as much asynchronous work off the request's critical path as possible.

Synchronous work often takes the form of writing data to a service's own domain. In a typical CRUD based web app, database writes usually need to happen synchronously along the critical path of an API request. If our app receives a `POST /user`  (triggering the creation of a new user account), we usually must create the user in our database before we can actually log them into the app. Though database updates commonly fall along the synchronous path, this is not always true — it will depend on the specific needs of your application. 

Asynchronous work often involves making non-transactional updates to other services. This is especially true if your system employs an event based architecture. As an example, assume you have an `Organization` service that manages all the information related to customer organizations (contracts, members, etc.) and an `Auth` service which manages authentication related things (login, SSO connections, etc.). Furthermore, let's assume that certain organizations allow custom SSO options for their members. When the `Organization` services receives a  
```
PATCH /org/1 
{ 
	"op": "replace", 
	"path": "/allow-custom-sso", 
	"value": false
}
```
it knows that it must remove the custom SSO option from that organization, but that's not all that needs to happen. Since the  `Auth` service manages the custom SSO connections, it needs to invalidate the custom SSO connections for all the members in that organization. We could iterate over all of an organization's members and make an API call to the `Auth` service to perform the unlinking but if there are a lot of members, that would be very expensive. We could improve the situation by creating a batch endpoint on the `Auth` service that accepts a list of members to unlink but depending on the product requirements, there may be a better solution.  Is it really essential that all users are unlinked from their custom SSO **before** you inform the end user that their settings update was successful? Maybe — maybe not. If no, a better option would be to put the unlinking step on an asynchronous path. 

The best way to implement asynchronous work depends on your programming language and surrounding infrastructure. However, one generally good option is to put asynchronous tasks onto a "work queue", and have a pool of "workers" read from the queue and perform whatever tasks they find. A nice description of this pattern can be found [here](https://docs.microsoft.com/en-us/azure/architecture/guide/architecture-styles/web-queue-worker). This architecture allows you to minimize latency on your API's critical paths and also provides fault tolerance for your asynchronous work.

The main take-away is that when writing APIs, you should carefully consider if the needed work is synchronous or asynchronous in nature. By trying to keep the unit of synchronous work as small as possible, you'll achieve better performance and can even make your code easier to reason about.