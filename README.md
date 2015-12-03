# blog-microservices

Source code for blogs regarding microservices at http://callistaenterprise.se/blogg/teknik/

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

