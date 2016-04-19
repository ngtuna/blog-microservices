# Pattern: API Gateway
## Context
You are building a microservices application. You need to develop multiple versions of the product details user interface:
- HTML5/JS-based UI for desktop and mobile browsers.
- Native Android & iOS clients - interact with server via REST APIs.
In addition, the application must expose REST-based API for use by 3rd party applications.
## Problem
How do the clients of a microservices-based application access the individual services?
# Forces / Constraints
- Microservices typically provide fine-grained APIs, which means that clients need to interact with multiple services to get the needed information.
- Different clients need different data.
- Network performance is different for different types of clients.
- The number of service instances and their locations (host+port) changes dynamically.
- Partitioning into services can change over time and should be hidden from clients.
## Solution
Implement an API gateway that is the single entry point for all clients. The API gateway handles requests in one of two ways. Some requests are simply proxied/routed to the appropriate service. It handles other requests by fanning out to multiple services.

Rather than provide a one-size-fits-all style API, the API gateway can expose a different API for each client. For ex, the Netflix API gateway runs client-specific adapter code that provides each client with an API that's best suited to it's requirements.

The API gateway might also implement security, e.g. verify that the client is authorized to perform the request.
## Resulting Context
**Benefits**:
- Insulates the clients from how the application is partitioned into microservices.
- Insulates the clients from the problem of determining the locations of service instances.
- Provides the optimal API for each client.
- Reduces the number of requests/roundtrips. For ex, the API gateway enables clients to retrieve data from multiple services with a single round-trip. Fewer requests also means less overhead and improves the user experience. An API gateway is essential for mobile applications.
- Simplifies the client by moving logic for calling multiple services from the client to API gateway.
**Drawbacks**:
- Increased complexity - the API gateway is yet another moving part that must be developed, deployed and managed.
- Increased response time due to the additional network hop through the API gateway - however, for most applications the cost of an extra roundtrip is insignificant.
