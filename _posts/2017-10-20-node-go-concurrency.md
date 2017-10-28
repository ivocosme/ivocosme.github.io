---
layout: post
title: It's not about the main... it's about not blocking the message.
description: ""
modified: 2017-10-20
tags: [Node, Go]
categories: [Node, Go]
---

## Quick Summary

This post is a personal opinion on how Golang is an awesome language. It seems to address several common issues / criticisms of top programming languages and, to me, it seems to have integrated the best features of each of them.

## Introduction  
One of the features that I like the most in Golang its how it was designed considering the concurrency world that we live in. CPUs now have lots of cores, with lots of threads, but few are the languages that really take advantage of these features (Most of them were created before these kind of CPUs reached the mass market). 

With this post, I’m trying to show the simplicity of out-of-the-box Goroutines logic versus the not so friendly JavaScript Event Loop system that is prone to block if you aren’t careful...
<!--more-->

I know what you’re saying… _What is this guy talking about? Is he dumb? There are callbacks, Promises, awaits…_ Still those were things that were ADDED to the language, and not from the very beginning (except for the callbacks, but you know what callback hell is right?!).  
 
**Important**: I've chosen NodeJS to "run up against" Go, because I work with it on a daily basis. It is NOT intended to say that one language is better or worst than the other.

## Requirements

* Node - 8.4.0
* npm - 5.4.2
* Go - 1.9
* curl - 7.54.0

## So... What are we here for?

In a more technical view, the purpose of this post is to show how both blocking / non-blocking scenarios work in Node (Javascript) and Go (Well... Go... lol).

There are three "runnable" examples:

* A NodeJS non-blocking scenario, where a setTimeout() function (a simple function that runs in async mode) does NOT block the event loop;
* A NodeJS blocking scenario, where the first request does some "expensive operation" that, by default, uses the main thread and blocks it
* A Go example where, by default and out-of-the-box, a GoRoutine is created to process the request.

## Setting up everything

### Setup assumptions

You already have Node and Go installed (see above the versions I've used).

* [Node download/installation guide](https://nodejs.org/en/download/package-manager/)
* [Go download/installation guide](https://golang.org/doc/install)

### Download the project

Open a terminal window inside and run:

```bash
git clone https://github.com/ivocosme/node-go-comparison.git
```

If you enter the new folder, you should see two folders with the following structure

```bash
.
├── GoExample
│   └── main.go
└── NodeExample
    ├── main.js
    └── package.json
```

### Setting up the project

For the sake of simplicity, we will:

* Use [concurrently](https://www.npmjs.com/package/concurrently), a NodeJS library that allows us to run two commands in parallel;
* cURL, so that we can access the API using the terminal.

To install concurrently, just run from the terminal

```bash
npm install -g concurrently
```

For cURl, you should have some version installed... Try to access it by opening the terminal and running

```bash
curl --version
```

You should see an output similar to this

```bash
curl 7.54.0 (x86_64-apple-darwin16.0) libcurl/7.54.0 SecureTransport zlib/1.2.8
Protocols: dict file ftp ftps gopher http https imap imaps ldap ldaps pop3 pop3s rtsp smb smbs smtp smtps telnet tftp
Features: AsynchDNS IPv6 Largefile GSS-API Kerberos SPNEGO NTLM NTLM_WB SSL libz UnixSockets
```

### Setting up the node example

Node application will require a small setup, since no dependencies were included in the repository. We'll have to download and install those dependencies (automatically) using npm.

So, on the root of our cloned folder, open a terminal window and run the following commands

```bash
cd NodeExample
npm install
```

You're now good to go!

## Up and Running

### Running the node server example

```bash
cd NodeExample
node .
```

### Running the go server example

```bash
cd GoExample
go run main.go
```

## Executing examples

### NodeJS non blocking example

Open a terminal window and run

```bash
concurrently -r "curl http://localhost:3000/non-blocking-eventloop" "curl http://localhost:3000/non-blocking-eventloop"
```

#### Expected result

Both messages should appear after 5 seconds (because both run async in parallel).

### NodeJS blocking example

Open a terminal window and run

```bash
concurrently -r "curl http://localhost:3000/blocking-eventloop" "curl http://localhost:3000/blocking-eventloop"
```

#### Expected result

The first message should appear after 5 seconds and the second message should appear 5 seconds after the first (because the first one took over the main thread).

### Go example

Open a terminal window and run

```bash
concurrently -r "curl http://localhost:8080/" "curl http://localhost:8080/" "curl http://localhost:8080/" "curl http://localhost:8080/" "curl http://localhost:8080/"
```

#### Expected result

All messages will appear randomly (because they have a random timer between 1 and 5 seconds) before the first 5 seconds. (Just like the non-blocking example using timeout).

## Conclusions

The way Go was build allows it to take "natural" advantage of the available cores and not "just block" the server when running. 

IMHO, this is a major advantage because it seems that the creators really took the "lessons learned" from other languages and integrated them in Go, making it a very promising (already proven!) language.

> If JavaScript, Java and C++ had an awesome super powered baby, it would be called Golang!
 
## References and further reading 
 
* A more technical comparison of Node and Go threading models - [https://edneypitta.com/on-node-go-concurrency/]()
* Ryan Dahl interview on his thoughts of NodeJS (Server-side) from large server side applications - [https://www.mappingthejourney.com/single-post/2017/08/31/episode-8-interview-with-ryan-dahl-creator-of-nodejs/]()
* The example that made me think about wrinting this post [https://gist.github.com/ghaiklor/9682b79353aade8a1e59]()