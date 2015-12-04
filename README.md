# blog-microservices

Source code for blogs regarding microservices at http://callistaenterprise.se/blogg/teknik/

# Required components
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
    * It will act as a OAuth Resource Server
    * It will pass through the OAuth Access Tokens that comes in the extern request to the API services
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

# Circuit Breaker and Monitoring
## Circuit Breaker: Netflix Hystrix
Netflix Hystrix provides circuit breaker capabilities to a service consumer. If a service doesn’t respond (e.g. due to a timeout or a communication error), Hystrix can redirect the call to an internal fallback method in the service consumer. If a service repeatedly fails to respond, Hystrix will open the circuit and fast fail (i.e. call the internal fallback method without trying to call the service) on every subsequent call until the service is available again. To determine wether the service is available again Hystrix allow some requests to try out the service even if the circuit is open. Hystrix executes embedded within its service consumer.

## Monitoring: Netflix Hystrix dashboard and Turbine
Hystrix dashboard can be used to provide a graphical overview of circuit breakers and Turbine can, based on information in Eureka, provide the dashboard with information from all circuit breakers in a system landscape.

## Source code walkthrough
### Gradle dependencies
Since Hystrix use `RabbitMQ` to communicate between circuit breakers and dashboards we also need to setup dependencies for that as well. For a service consumer, that want to use Hystrix as a circuit breaker, we need to add:
```
compile("org.springframework.cloud:spring-cloud-starter-hystrix:1.0.0.RELEASE")
compile("org.springframework.cloud:spring-cloud-starter-bus-amqp:1.0.0.RELEASE")
compile("org.springframework.cloud:spring-cloud-netflix-hystrix-amqp:1.0.0.RELEASE")
```
For a complete example see `product-composite-service/build.gradle`.
To be able to setup an Turbine server add the following dependency:
```
compile('org.springframework.cloud:spring-cloud-starter-turbine-amqp:1.0.0.RELEASE')
```
For a complete example see `turbine/build.gradle`.
### Infrastructure servers
Set up a `Turbine` server by adding the annotation `@EnableTurbineAmqp` to a standard Spring Boot application:
```
@SpringBootApplication
@EnableTurbineAmqp
@EnableDiscoveryClient
public class TurbineApplication {

    public static void main(String[] args) {
        SpringApplication.run(TurbineApplication.class, args);
    }

}
```
For a complete example see `TurbineApplication.java`.
To setup a Hystrix Dashboard add the annotation `@EnableHystrixDashboard` instead. For a complete example see `HystrixDashboardApplication.java`.
### Business services
To enable Hystrix, add a `@EnableCircuitBreaker` annotation to your Spring Boot application. To actually put Hystrix in action, annotate the method that Hystrix shall monitor with `@HystrixCommand` where we also can specify a fallback-method, e.g.:
```
@HystrixCommand(fallbackMethod = "defaultReviews")
public ResponseEntity<List<Review>> getReviews(int productId) {
    ...
}

public ResponseEntity<List<Review>> defaultReviews(int productId) {
    ...
}
```
The fallback method is used by Hystrix in case of an error (call to the service fails or a timeout occurs) or to fast fail if the circuit is open. For a complete example see `ProductCompositeIntegration.java`.
