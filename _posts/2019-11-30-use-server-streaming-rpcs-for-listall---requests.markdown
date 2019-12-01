---
title: Use Server Streaming RPC's for ListAll__() Requests
layout: post
categories: [today-i-learned]
excerpt_separator: <!--more-->
---

*Originally written: November 27, 2019*

While investigating an issue with one of our internal services, I came across the following error message:
```
…(rpc error: code = ResourceExhausted desc = grpc: received message larger than max (8985920 vs. 8388608)) at … 
```
<!--more-->
Probably because of some CS exam I had back in college, I immediately noticed that 8388608 bytes is exactly 8MiB. A number that round was certainly hard coded somewhere, and the rest of the error message told me what it was for: the max allowed GRPC message size. 

Sure enough, after looking a little deeper I found the spot in our code where we specified the GRPC message size limit.
```
MaxCallRecvMsgSize: 8 * 1024 * 1024, // 8MiB
```
Looking at the `git blame` I was able to determine that the person who explicitly set the value to 8MiB did so because they were fixing an error where the message size was greater than the previous default (set by GRPC) of 4MiB. As a quick fix to this issue (time was of the essence), I increased the `MaxCallRecvMsgSize` limit to 12MiB but I resolved to implement a better long term solution once I had some free time.

The RPC which was leading to these large message sizes had the form `ListAll__()`.  The client issuing this call was fetching the entire collection of some database entity so that it could do some offline processing. That entity count used to be small enough to fit into a single 4MiB payload but as we grew as a company, so did the entity count. 4MiB was no longer enough, now we needed at least 9MiB. 

Besides running into the GRPC message size limits, transmitting large payloads like this via RPC places a large memory burden on both the client and the server. Before sending the response, the server needs to collect the entire result set into memory before it can serialize, and send it over the wire. Similarly, the client needs to wait to collect the entire response before it can begin processing any of it. 8MiB of memory per request can really add up quickly. 

Using a Server Streaming RPC helps solve for both of these issues. Rather than sending back a single huge response, the server will send back a stream of results. This enables all of the data to be transferred without any individual message exceeding a reasonable message size. It also allows both the client and the server to process a chunk of the data at a time. This allows the server better control of its own memory footprint since it can determine how to chunk the responses and it provides the client with the option to process results as they come in.

For the particular `ListAll__()` RPC we were using, migrating over to a Server Streaming RPC is a great long term solution (particularly because we expect the response size to continue growing).

