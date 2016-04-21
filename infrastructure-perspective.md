# Infrastructure perspective - Required components

- Service Discovery server: internal DNS
- Dynamic Routing & Load Balancer:
- Circuit Breaker: avoid the chain of failures problem
- Monitoring: monitor state and collect run time statistics
- Centralized log analysis:
    + centralized log analysis function
    + reaching out to the servers and collect the log-files that each micro-services produce
- Edge server / API gateway
    + expose the API services externally
    + prevent unauthorized access
- Authentication server
    + protect the exposed API services

Others requirements:
- Database per services
- multiple services per host/vm
