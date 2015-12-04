# blog-microservices

Source code for blogs regarding microservices at http://callistaenterprise.se/blogg/teknik/

# REQUIRED COMPONENTS
Central Configuration server
Instead of a local configuration per deployed unit (i.e. microservice) we need a centralized management of configuration. We also need a configuration API that the microservices can use to fetch configuration information.

### Service Discovery server
Instead of manually keeping track of what microservices that are deployed currently and on what hosts and ports we need service discovery functionality that allows, through an API, microservices to self-register at startup.

### Dynamic Routing and Load Balancer
Given a service discovery function, routing components can use the discovery API to lookup where the requested microservice is deployed and load balancing components can decide what instance to route the request to if multiple instances are deployed for the requested service.

### Circuit Breaker
To avoid the chain of failures problem we need to apply the Circuit Breaker pattern, for details see the book Release It! or read the blog post Fowler - Circuit Breaker.

### Monitoring
Given that we have circuit breakers in place we can start to monitor their state and also collect run time statistics from them to get a picture of the health status of the system landscape and its current usage. This information can be collected and displayed on dashboards with possibilities for setting up automatic alarms for configurable thresholds.

### Centralized log analysis
To be able to track messages and detect when they got stuck we need a centralized log analysis function that is capable to reaching out to the servers and collect the log-files that each microservice produce. The log analysis function stores this log information in a central database and provide search and dashboard capabilities.
Note: To be able to find related messages it is very important that all microservices use correlation id’s in the log messages.

### Edge Server 
To expose the API services externally and to prevent unauthorized access to the internal microservices we need an edge server that all external traffic goes through. An edge server can reuse the dynamic routing and load balancing capabilities based on the service discovery component described above. The edge server will act as a dynamic and active reverse proxy that don’t need to be manually updated whenever the internal system landscape is changed.

### OAuth 2.0 protected API’s
To protect the exposed API services the OAuth 2.0 standard is recommended. Applying OAuth 2.0 to the suggested solution results in:

* A new component that can act as a OAuth Authorization Server
* The API services will act as OAuth Resource Server
* The external API consumers will act as OAuth Clients
* The edge server will act as a OAuth Token Relay meaning:
** It will act as a OAuth Resource Server
** It will pass through the OAuth Access Tokens that comes in the extern request to the API services
Note: Over time the OAuth 2.0 standard will most probably be complemented with the OpenID Connect standard to provide improved authorization functionality.

# Eureka, Ribbon and Zuul
## Source code walkthrough
Each microservice is developed as standalone `Spring Boot application` and uses `Undertow`, a lightweight Servlet 3.1 container, as its web server. `Spring MVC` is used to implement the REST based services and `Spring RestTemplate` is used to perform outgoing calls.

### Gradle dependencies
To use `Eureka` and `Ribbon` in a microservice to register and/or call other services simply add the following to the build file:
```
compile("org.springframework.cloud:spring-cloud-starter-eureka:1.0.0.RELEASE")
```
For a complete example, check out `product-service/build.gradle` or `discovery-server/build.gradle`
### Infrastructure servers
For a `Eureka` server add the annotation `@EnableEurekaServer` to a standard Spring Boot application:
```
@SpringBootApplication
@EnableEurekaServer
public class EurekaApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaApplication.class, args);
    }
}
```
To bring up a `Zuul` server you add a `@EnableZuulProxy` - annotation instead. Check out `ZuulApplication.java`
With these simple annotations you get a default server configurations that gets yo started. When needed, the default configurations can be overridden with specific settings. One example of overriding the default configuration is where we have limited what services that the edge server is allowed to route calls to. By default Zuul set up a route to every service it can find in Eureka. With the following configuration in the `application.yml` - file we have limited the routes to only allow calls to the composite product service:
```
zuul:
  ignoredServices: "*"
  routes:
    productcomposite:
      path: /productcomposite/**
```
### Business services
To auto register microservices with `Eureka`, add a `@EnableDiscoveryClient` - annotation to the Spring Boot application.
```
@SpringBootApplication
@EnableDiscoveryClient
public class ProductServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(ProductServiceApplication.class, args);
    }
}
```
For a complete example see `ProductServiceApplication.java`
To lookup and call an instance of a microservice, use `Ribbon` and for example a Spring RestTemplate like:
```
@Autowired
private LoadBalancerClient loadBalancer;
...
public ResponseEntity<List<Recommendation>> getReviews(int productId) {

        ServiceInstance instance = loadBalancer.choose("review");
        URI uri = instance.getUri();
...
        response = restTemplate.getForEntity(url, String.class);
```
The service consumer only need to know about the name of the service (`review`), `Ribbon` (i.e. the LoadBalancerClient class) will find a service instance and return its URI to the service consumer.
For a complete example see `Util.java` and `ProductCompositeIntegration.java`

