# Pattern: Data management

## Database per service
### Context
You are developing microservices application. Most services need to persist data in some kind of database.
### Problem
What's the database architecture in a microservices application?
### Forces / Constraints
- services must be loosely coupled so that they can be developed, deployed and scaled independently
- some business transactions need to update data that is owned by multiple services.
- some queries must join data that is owned by multiple services.
- databases must sometimes be replicated and shared in order to scale.
- different services have different data storage requirements. For some services, a relational database is best choice. Others might need a NoSQL (MongoDB) database or graph (Neo4J)
### Solution
Keep each microservice's persistent data private to that service and accessible only via its API. The service's database is effectively part of the implementation of that service. It cannot be accessed directly by other services.

However, there are a few different ways to keep a service's persistent data private. You do not need to provide a separate database server for each service. For ex, if you are using a relational database, then the options are:
- private tables per service: each service owns a set of tables that must only be accessed by that service.
- schema per service: each service has a database schema that is private to that service.
- database server per service: ehh...
### Resulting Context
**Benefits**
- helps ensure that services are loosely coupled.
- each service can use the type of database that is best suited to its needs.
**Drawbacks**
- implementing business transactions that span multiple services is not straightforward. The best solution is to use an eventually consistent, event-driven architecture. Services publish events when they update data. Others subscribe to events and update their data in response.
- implementing queries that join data is challenging. There are some solutions:
  + Application-side joins: application performs the join rather than the database.
  + Command Query Responsibility Segregation (CQRS)

## Shared database
Use a (single) database that is shared by multiple services.

## Event-driven architecture
Maintain data consistency across microservices by exchanging events.

## Event sourcing
Reliably publish events whenever state changes by using Event Sourcing. Event Sourcing persists each business entity as a sequence of events, which are replayed to reconstruct the current state.

## CQRS
Split the system into two parts. The command side handles create, update and delete requests. The query side handles queries using one or more materialized views of the application's data.

## transaction log tailing
Reliably publish events whenever state changes by tailing the transaction log.

## Database trigger
Reliably publish events whenever state changes by using database triggers. Each trigger inserts an event into an `EVENTS` table, which is polled by a separate process that publishes the events.

## Application events
Reliably publish events whenever state changes by having the application insert events into an `EVENTS` table as part of the local transaction. A separate process polls the `EVENTS` table and publishes the events to a message broker.
