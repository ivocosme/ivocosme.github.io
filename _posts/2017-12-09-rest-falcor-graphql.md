---
layout: post
title: Why are you RESTing when you can be up and querying?
description: "This post brings a quick (non-exhaustive) glance of two approaches (Falcor & GraphQL) that can leverage your game acessing your data exposed through an API."
modified: 2017-12-09
tags: [Node, Falcor, GraphQL]
categories: [Node, GraphQL]
---

## Quick Summary

Almost everyone recognizes the value of [REST](https://en.wikipedia.org/wiki/Representational_state_transfer) architecture implementations on the Internet nowadays. But if you use it on a daily basis, you know that it lacks that "extra mile" that would make your projects work even better.
 
This post brings a quick (non-exhaustive) glance of two approaches (Falcor & GraphQL) that can leverage your game acessing your data exposed through an API. Dependening on your use case, any one of them can be a perfect fit for your next project.

## Introduction  

There are two new kids on the block. Well... they're not really new, but they've only become "hype" lately. I'm talking about [Falcor](http://netflix.github.io/falcor/) and [GraphQL](http://graphql.org/).

If you have the luck of not being stuck in a SOAP-like world, you're probably familiar working with REST. And, in that case, this post might interest you!

<!--more-->

GraphQL was created by Facebook and is described as "(...) _a query language for APIs and a runtime for fulfilling those queries with your existing data. (...)_".

Falcor on the other hand, was designed by Netflix and is seen by its creators as "_a JavaScript library for efficient data fetching_".

One is a library and the other is a query language... is it fair to put both on the same stack? I think that they both work very well _as a way to gather the information_ your frontends need in a very performant and optimized way.

### ** Important ** 

* When I say "non-exhaustive glance" about this post, I mean that there are LOTS of features that these tools have, that are not even mentioned here. Example: POST operations in REST, 'set' operations in Falcor, Mutations in GraphQL.

* All running examples will end up with the same result. Another purpose of this post is to show you how different approaches will work, but can optimize and simplify your solution.

* Do not focus too much on the database data schema, but on how the information is passed and retrieved into the frontend.

* This post is just a PoC, it does not address issues like "You should only have a secure entry point to your Backend APIs", or "You should have something that manages your authentication / authorization", which in real world use cases MUST be taken into account.

Let's get to work then!

## Scenario

Starting from a simple everyday use case, I'll try to show you three different approaches (REST + Falcor + GraphQL) with two different targets (Mobile + Web).

This is a simple scenario where you have some customers that buy you stuff, and you have that information stored in some database. For security reasons, you have the information (customers and invoices) split in two databases and that information is stored behind two different APIs.

Also, this scenario tries to demonstrate a "Web approach" that makes use of the existing Javascript libraries to help fetch data; and a "Mobile approach", that uses makes a direct HTTP call to the server (without any library in between) requesting the information.

### Use case

The use case is a simple situation where the user would want to see his/her information and also a list of the invoices he/she has.

### REST

Both web and mobile versions of the REST solution run on the server that is also serving both pages. In fact, both approaches make use of [axios](https://github.com/axios/axios) an HTTP client for requests made from the browser.

In this example, both solutions are in charge of making these HTTP requests, build the JSON response and then render it on the page.

<figure>
	<a href="{{ site.url }}/images/api-approach-rest.png"><img src="{{ site.url }}/images/api-approach-rest.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/api-approach-rest.png" title="A possible approach accessing API layer from the client">A possible approach accessing API layer from the client</a>.</figcaption>
</figure>

#### Problems

The most obvious problem is the N+1 problem. Because we don't know (yet) how many invoices will the user have, we have to make the first request (1), and then we have to make a request for every retrieved invoice identifier (N), to fetch their details. 

**Note**: This problem won't be addressed here, but there are good solutions that help mitigate this problem (e.g. Facebook's [DataLoader](https://github.com/facebook/dataloader) utility library for Nodejs).

#### Possible solutions

A possible solution is to use a caching utility library like [DataLoader](https://github.com/facebook/dataloader) to minimize the number of requests. You can also think on the solution I'll mention in the next topic... or... You can try Falcor!

###### To think later...

Since all your business logic still lives in the APIs, and you are not blocking your event loop while you are accessing network, would it be worth it to have some other server just as an info aggregation tool instead of doing it directly from the page?

### Falcor

Falcor uses a model where it exposes a virtual JSON (in a server-side app) that is hosted under /model.json.

In a straightforward  way, in the web approach (that uses the library), developers use a Javascript-like syntax to structure the data they want to receive, and then send it to this endpoint.

On the other hand, the mobile approach uses an HTTP request passing all the arguments. This would apply to situations where the mobile apps were developed, for instance, natively.

As mentioned, this approach needs a Falcor Router server app that must be exposed for the clients to request data.

<figure>
	<a href="{{ site.url }}/images/api-approach-falcor.png"><img src="{{ site.url }}/images/api-approach-falcor.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/api-approach-falcor.png" title="A possible approach accessing API layer using Falcor to extract the info.">A possible approach accessing API layer using Falcor to extract the info</a>.</figcaption>
</figure>

#### Problems

##### Very much focused in a full javascript stack

FalcorJS was developed with Javascript in mind, so, for web apps, it is perfectly fine. The problem starts, when you have mobile apps that weren't wrote in Javascript (_Blasfemy!!!_ you say, but yes... there are mobile apps written in other languages than Javascript :D ).

##### Let the developer choose!

Falcor was developed with an assumption in mind:

>  _You have an array of objects... Today it has a few entries... tomorrow will have thousands... So let's just block you right there, and force you to be selective._ 

"_Rather than allow you to retrieve an entire Object, Models force you to be explicit and retrieve only those values needed in a given scenario. Similarly, when displaying an Array of items, Models do not allow you to retrieve the entire Array upfront._"

Even though I consider their mindset correct, I think it would be fair to let the developer decide if the data should be retrieved or not.

##### Lacks non-javascript support and libraries

Another problem with this approach, is the lack of a "non-JavaScript" library that can be used with "non-JavaScript" mobile apps (_Blasfemy again!!!_ :D). **Detailed technical explanation**: To bypass this "lack", you can just use the library in a browser, open the dev tools in the Network tab and then copy the query parameters to make a direct HTTP request to the Falcor Router. The problem, is that the order that was set with the library, isn't the same as using the "direct connection".

#### Possible solutions

To overcome the objects limitation, you can take advantage of the "Atom" approach. "_Atoms can be used to get around the restriction against retrieving JSON Objects and Arrays. Let’s say that we have an Array which we are certain will remain small, like a list of video subtitles for example. By boxing the subtitles Array in an Atom, we are able to retrieve the entire Array using the abstract get operation._"

For the non-JavaScript library, I think that a possible workaround is to use some wrapper class that transforms the fields into query parameters and then organizes them on the return.

### GraphQL

GraphQL uses a query logic to access data. It doesn't matter if you are in a Web or a Mobile version. You just create your functions, add some conditions (e.g. "Please send me this data if the query parameter I'm sending is true" - You can have different results for the same request, just by setting different parameters!), say what info you want to receive and in what order and... done! You get your data structured just like you requested.

Very similar to Falcor, but IMO in a much more flexible way!

<figure>
	<a href="{{ site.url }}/images/api-approach-falcor.png"><img src="{{ site.url }}/images/api-approach-graphql.png" alt=""></a>
	<figcaption><a href="{{ site.url }}/images/api-approach-graphql.png" title="A possible approach accessing API layer using GraphQL to extract the info.">A possible approach accessing API layer using GraphQL to extract the info</a>.</figcaption>
</figure>

## Requirements

* Node - 8.4.0+
* npm - 5.4.2+

## Setting up everything

### Setup assumptions

You already have Node installed (see above the versions I've used).

* [Node download/installation guide](https://nodejs.org/en/download/package-manager/)

### Download the project

Open a terminal window inside and run:

```bash
git clone https://github.com/ivocosme/rest-falcor-graphql.git
```

If you enter the new folder, you should see two folders with the following structure

```bash
.
├── .eslintrc
├── .gitignore
├── .jshintrc
├── app
│   ├── index.js
│   └── public
├── package.json
├── servers
│   ├── common
│   ├── falcor-approach
│   └── graphql-approach
└── services
    ├── customers
    └── invoices
```

### Setting up the project

For the sake of simplicity, we will:

* Use [concurrently](https://www.npmjs.com/package/concurrently), a NodeJS library that allows us to run two commands in parallel.

To install concurrently, just run from the terminal

```bash
npm install -g concurrently
```

## Up and Running

### What will I run?

Executing the command from next topic will run:

* Run the customers (database) API in port 50000
* Run the invoices (database) API in port 50001
* Run the Falcor Router in port 40000
* Run the GraphQL Runtime in port 40001
* Run the Frontend page in port 3000

### Running the example

Open a terminal window and run

```bash
npm start
```

**Important:** Running this command is the same as running

```bash
CLIENT_PORT=3000 FALCOR_PORT=40000 GRAPHQL_PORT=40001 CUSTOMERS_PORT=50000 INVOICES_PORT=50001 concurrently \"node services/customers/index.js\" \"node services/invoices/index.js\"  \"node servers/falcor-approach/index.js\" \"node servers/graphql-approach/index.js\" \"node app/index.js\"
```

Then, open the browser [to see the result](http://localhost:3000).

## What am I seeing here?

You are currently seeing the result of making the request to see the user info (Id: 1) and its invoices (Ids: 10000/1/2).

If you look closely, you'll see that there are specific fields that should only be retrieved by the "web version" and some other fields that are retrieved by the "mobile version".

### REST

REST approach has two files:

```bash
.
├── app
│   └── public
│       └── rest-web.html
│       └── rest-mobile.html
```
The first one has logic for the web approach, and the second one, has logic for the mobile approach.

Both approaches retrieve all user info. Then, for each approach, the developer must filter all unwanted fields.

Also, we did not know the invoice ids to request the information right on the first request. We then have to make a single request for each invoice identifier to retrieve it.

#### Web

As mentioned above, we start by requesting user details,

```javascript
axios.get('http://localhost:50000/1')
```
we remove all "mobile-specific" data from the answer,

```javascript
_.omit(originarCustomer, ['extraInfoMobile1', 'extraInfoMobile2', 'extraInfoMobile3']);
```

then, for each invoice id, we go fetch server data (we are adding it into an array of promises to resolve them faster).

```javascript
_.forEach(originarCustomer.invoices, (invoiceId) => {
    invoicePromises.push(axios.get('http://localhost:50001/'+invoiceId))
});
axios.all(invoicePromises)
```

After merging the response

```javascript
customer.invoices = invoices;
```

We write everything to the document

```javascript
document.write(JSON.stringify(customer));
```

#### Mobile

As mentioned above, we start by requesting user details,

```javascript
axios.get('http://localhost:50000/1')
```
we remove all "mobile-specific" data from the answer,

```javascript
_.omit(originarCustomer, ['extraInfoWeb1', 'extraInfoWeb2', 'extraInfoWeb3']);
```

then, for each invoice id, we go fetch server data (we are adding it into an array of promises to resolve them faster)

```javascript
_.forEach(originarCustomer.invoices, (invoiceId) => {
    invoicePromises.push(axios.get('http://localhost:50001/'+invoiceId))
});
axios.all(invoicePromises)
```

After merging the response

```javascript
customer.invoices = invoices;
```

We write everything to the document

```javascript
document.write(JSON.stringify(customer));
```

### Falcor

Falcor approach uses two files

```bash
.
├── app
│   └── public
│       └── index.html
│       └── falcor-web.html
```

The first one, for the web approach, it uses the 'falcor-web.html' file to take advantage of the falcor library. The mobile approach, since it tries to simultate a non-javascript project, uses a direct HTTP request in 'index.html', and sets the content into an [iframe](https://www.w3schools.com/tags/tag_iframe.asp).

#### Web

When using the falcor library, the developer sets the model location

```javascript
new falcor.Model({source: new falcor.HttpDataSource('http://localhost:40000/model.json?customer=1') });
```

and then, it requests the information he wants

```javascript
get('customer["customerId","email","name","address","extraInfoWeb1","extraInfoWeb2","extraInfoWeb3","invoices"]')
```

**Important** As mentioned above, it is not recommended, nor is it possible *using best practices* to access arrays/ objects directly (we are requesting the list of invoices). To achieve this, we use an [Atom](http://netflix.github.io/falcor/documentation/jsongraph.html#new-primitive-value-types).

#### Mobile

In case we have to access Falcor Router using a direct HTTP request, we just have to set the required fields as query parameters

```
http://localhost:40000/model.json?customer=1&paths=%5B%5B%22customer%22%2C%5B%22address%22%2C%22customerId%22%2C%22email%22%2C%22extraInfoWeb1%22%2C%22extraInfoWeb2%22%2C%22extraInfoWeb3%22%2C%22invoices%22%2C%22name%22%5D%5D%5D&method=get
```

**Note**: The main issue with this approach is that, even though all information is retrieved, the order is not guaranteed as when accessing it directly.

### GraphQL

Regarding GraphQL, the access is made using a direct HTTP access with query parameters for both web and mobile approach. Both requests are made from the 'index.html' file.

```bash
.
├── app
│   └── public
│       └── index.html
```

It's important to mention that these query parameters take all the needed information. The structure for web and mobile approaches is *the same*. However, both take some metadata to assert if the value should be retrieved (or not) according to the variable parameters.

Both approaches use an [iframe](https://www.w3schools.com/tags/tag_iframe.asp) to render the retrieved contents.


#### Web

The web approach makes the same request as the mobile approach, by telling that web params should be included and the mobile params should be excluded

```html
extraInfoMobile1 @include(if: $isMobile),
extraInfoMobile2 @include(if: $isMobile),
extraInfoMobile3 @include(if: $isMobile),
extraInfoWeb1 @skip(if: $isMobile),
extraInfoWeb2 @skip(if: $isMobile),
extraInfoWeb3 @skip(if: $isMobile),
```

if the variables say so

```html
variables={
  "customerId": "1",
  "isMobile": false 
}
```

#### Mobile

The mobile approach makes the same request as the web approach, by telling that mobile params should be included and the web params should be excluded

```html
extraInfoMobile1 @include(if: $isMobile),
extraInfoMobile2 @include(if: $isMobile),
extraInfoMobile3 @include(if: $isMobile),
extraInfoWeb1 @skip(if: $isMobile),
extraInfoWeb2 @skip(if: $isMobile),
extraInfoWeb3 @skip(if: $isMobile),
```

if the variables say so

```html
variables={
  "customerId": "1",
  "isMobile": true 
}
```

## Conclusions

I think that, in the near future, REST won't be massively substituted for neither of these solutions.

Both Falcor and GraphQL bring a new fresh way to access your data in a more structured and business directed way.

However, IMO, the fact that Falcor _forces_ users to develop their applications using a more limited approach is a turn down which makes me like GraphQL more.

If you are starting a new project and have some space to inovate, you should give a try to GraphQL as your API aggregator and single point of access to your data.
 
## References and further reading 
 
* REST - [https://en.wikipedia.org/wiki/Representational_state_transfer]()
* FalcorJS - [http://netflix.github.io/falcor/]()
* FalcorJS Tutorial Videos - [http://netflix.github.io/falcor/video-tutorials/introduction-to-the-model.html]()
* Introduction to GraphQL - [http://graphql.org/learn/]()
* Discussion on Falcor vs. GraphQL - [https://news.ycombinator.com/item?id=13611263]()
* graphiql - In-browser GraphQL IDE [https://github.com/graphql/graphiql]()
* ProgrammableWeb article - [https://www.programmableweb.com/news/what-graphql-and-why-does-it-matter/how-to/2017/10/29]()
* Falcor Router example - [https://github.com/netflix/falcor-router-demo]()
* Github GraphQL documentation - [https://developer.github.com/v4/guides/intro-to-graphql/]()
* GraphQL working draft - [http://facebook.github.io/graphql/October2016/]()

