---
layout: post
title:  "Integration test rollback with Web API"
date:   2017-12-04 09:07:50 +0100
categories: csharp sqlserver integration testing
disqus: true
live: false
---

In this post we’ll discuss the use of Web API, SQL Server, `HttpServer` and `TransactionScope` to write easy to maintain and rerunnable integration tests.

The goal of this post is to get to a point whereby we can setup a test (add data to the database), run the test and restore the application back to its initial state.

The code below shows a test web api controller that calls a database and increments the value from the database.

(Code)
 
A test that calls the api to validate the returned value.

(Code)

Next we have a class than runs as a global setup before each test. This: 
1. Creates the `TransactionScope`
2. Creates an in process `HttpServer`
3. Creates a `HttpClient` that will delegate all calls via the `HttpServer`

The special part here is the use of an in memory/in process `HttpServer` as a `HttpMessageHandler` for the `HttpClient`.

So instead of the HttpClient making a call via the network it will call the `HttpServer` directly and it will direct the call in memory. This is important HttpServer I’m in scope of our `TransactionScope`. Meaning any `SqlConnection` will use the current scopes transaction.

So when we say rollback the transaction this will apply to all the SqlConnections made during HttpClients call. Rolling back all data changes.

To see the full code example check out my GitHub account [rolling back integration tests](http://github.com/roy46)