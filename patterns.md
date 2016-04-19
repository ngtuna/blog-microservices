# Pattern: Microservices Architecture

## Context

You are developing a service-side enterprise application with some common perspectives:
- support a variety of different clients
- expose APIs for 3rd parties to consume
- might also integrate with other applications via either web services or message broker
- handles requests (HTTP-based and messages) by
  + executing business logic
  + accessing database
  + exchanging messages with other systems
  + returning a HTML/JSON/XML response

Application's elements:
- Presentation: responsible for handling HTTP requests and responding HTML/JSON/XML
- Business logic: core business processing
- Database access logic: data access objects responsible for accessing the database
- Integration logic: messaging layer

## Problem
What's the application's deployment architecture?

## Forces / Constraints
- There is a team of developers working on the application
- New team members must quickly become productive
- Application must be easy to understand and modify
- You want to practice CD
- You must run multiple copies of the application on multiple machines in order to satisfy scalability and availability requirements
- You want to take advantage of emerging technologies (frameworks, programming languages, etc)

## Solution
- Architect the application by applying the `Scale Cube`. Specifically Y-axis (functional decomposition). Two ways of decomposing:
  + verb-based: define service implementing a single use case (ex: checkout)
  + noun-based: define service implementing all operations related to a particular entity (ex: customer management)
  Note: X-axis -> clone of app, Z-axis -> multiple servers run subsets of data.

- Functionally decompose application into a set of collaborating services.
- Each service implements a set of narrowly, related functions. (order management, customer management, etc)
- Services communicate using either synchronous protocols (HTTP/REST) or asynchronous protocols (AMQP)
- Services are developed and deployed independently
- Each service has its own database in order to be decoupled from other services.

## Resulting Context
**benefits**
- Each microservice is relatively small
  + easier to develop
  + IDE is faster making developers more productive
  + the web container starts faster, speed up deployment
- Each microservice can be deployed independently of others -> easier to adopt CD
- Easier to scale development by organizing dev effort around multiple teams.
- Improved fault isolation. Ex: problem will affect to single service. Other services will continue to handle requests.
- Eliminates any long-term commitments to a technology stack.

**drawbacks**
- Devs must deal with complexity of creating a distributed system
  + Tool/IDE might be oriented on building monolithic apps
  + Testing is more difficult
  + Devs must implement inter-service communication mechanism
  + New way of collaborating between teams
- Deployment complexity: many different service types.

Challenges:
- Decide how to partition the system into microservices.
