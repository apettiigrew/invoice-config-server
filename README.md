# Invoice Microservices App

The Invoice microservice app is a simple invoice management app built to demonstrate the Microservices Architecture Pattern using SpringBoot, Spring Cloud and Docker. The project is intended as a pure learning process to try out various concepts in a microservices and distributed environment. You are free to fork it and turn into something else or even provide cool updates via pull request. I'm happy to collab.

## Functional services

The Invoice System is composed into three core microservices. Each application is independently deployable and structured around specific business domains.

[add image here]

#### User service
Contains general logic to create/register a user to the open source Keycloak Identity Access and Management Server.

| Endpoint Name  | Method | URL                    | Description                      |
|---------------|--------|------------------------|----------------------------------|
| create-users  | POST   | `{{user_url}}`         | Create a new user account       |
| update-users  | PATCH  | `{{user_url}}/:id`     | Update user details             |
| get all users | GET    | `{{user_url}}?page=0&size=10` | Retrieve a paginated list of users |
| get single user | GET    | `{{user_url}}/:id`     | Retrieve details of a specific user |
| delete-users  | DELETE | `{{user_url}}/:id`     | Delete a user account           |


#### Invoice service
Performs the CRUD operations required to manage an invoice.

| Endpoint Name       | Method  | URL                          | Description                        |
|---------------------|--------|------------------------------|------------------------------------|
| create invoice     | POST   | `{{invoice_url}}`            | Create a new invoice              |
| get all invoices   | GET    | `{{invoice_url}}?page=0&size=10` | Retrieve a paginated list of invoices |
| get single invoice | GET    | `{{invoice_url}}/:id`        | Retrieve details of a specific invoice |
| update invoice     | PATCH  | `{{invoice_url}}/:id`        | Update details of an invoice      |
| delete invoice     | DELETE | `{{invoice_url}}/:id`        | Delete an invoice                 |
| contact-info       | GET    | `{{invoice_url}}/contact-info` | Retrieve invoice-related contact info |
| bus refresh       | POST   | `{{invoice_url}}/actuator/busrefresh` | Refresh configuration bus |


#### Notification service
Sends an email via SendGrid api whenever an invoice has been created, updated or deleted. This servers as eg of standalone service to manage all notification in the system. Can be extended to send sms, send reminder etc.
Currently this is possible as Spring Cloud function that receives event from a message broker and sends an email to the client who is the recipient of an invoice.


#### Notes
- The User service connects directly with they Keycloak Server where all user information exists and is managed. 
- Each microservice has its own database, so there is no way to bypass API and access persistence data directly.
- MySql is used as a primary database for each of the services.

## Infrastructure
[Spring cloud](https://spring.io/projects/spring-cloud) provides powerful tools for developers to quickly implement common distributed systems patterns
<img width="880" alt="Infrastructure services" src="https://cloud.githubusercontent.com/assets/6069066/13906840/365c0d94-eefa-11e5-90ad-9d74804ca412.png">

### Config service
[Spring Cloud Config](http://cloud.spring.io/spring-cloud-config/spring-cloud-config.html) is horizontally scalable centralized configuration service for the distributed systems. It uses a pluggable repository layer that currently supports local storage, Git, and Subversion. 

In this project, the config server connects with a github repository to pull configuration for the different microservices. You can see shared configuration repo here [invoice-config-server](https://github.com/apettiigrew/invoice-config-server). 

##### Spring Cloud Config Server & Spring Cloud Bus
With the Config Server you have a central place to manage external properties for applications across all environments. 

[Spring Cloud Bus](https://spring.io/projects/spring-cloud-bus), available at facilitates seamless communication between all connected application instances by establishing a convenient event broadcasting channel. It offers an implementation for AMQP brokers, such as RabbitMQ, and Kafka. Spring Cloud Config offers the Monitor library, which enables the triggering of configuration change events in the Config Service. By exposing the /monitor endpoint, it facilitates the propagation of these events to all listening applications via the Bus. The Monitor library allows push notifications from popular code repository providers such as GitHub, GitLab, and Bitbucket. 


### API Gateway

API Gateway is a single entry point into the system, used to handle requests and routing them to the appropriate backend service. 

Include the spring-cloud-starter-gateway, spring-cloud-starter-config & spring-cloud-starter-netflix-eureka-client maven dependencies.

Configure the properties: In the application properties or YAML file,

    cloud:  
      gateway:  
        discovery:  
          locator:  
            enabled: false  
            lowerCaseServiceId: true



### Service Discovery

[](https://github.com/sqshq/piggymetrics/blob/master/README.md#service-discovery)

Service Discovery allows automatic detection of the network locations for all registered services. These locations might have dynamically assigned addresses due to auto-scaling, failures or upgrades.

The key part of Service discovery is the Registry. In this project, we use Netflix Eureka. Eureka is a good example of the client-side discovery pattern, where client is responsible for looking up the locations of available service instances and load balancing between them.

With Spring Boot, you can easily build Eureka Registry using the  `spring-cloud-starter-eureka-server`  dependency,  `@EnableEurekaServer`  annotation and simple configuration properties.

### Monitor dashboard

[](https://github.com/sqshq/piggymetrics/blob/master/README.md#monitor-dashboard)

In this project configuration, each microservice with Hystrix on board pushes metrics to Turbine via Spring Cloud Bus (with AMQP broker). The Monitoring project is just a small Spring boot application with the  [Turbine](https://github.com/Netflix/Turbine)  and  [Hystrix Dashboard](https://github.com/Netflix-Skunkworks/hystrix-dashboard).

Let's see observe the behavior of our system under load: Statistics Service imitates a delay during the request processing. The response timeout is set to 1 second:

## Let's try it out

[](https://github.com/sqshq/piggymetrics/blob/master/README.md#lets-try-it-out)

Note that starting 8 Spring Boot applications, 4 MongoDB instances and a RabbitMq requires at least 4Gb of RAM.

#### Before you start

[](https://github.com/sqshq/piggymetrics/blob/master/README.md#before-you-start)

-   Install Docker and Docker Compose.
-   Change environment variable values in  `.env`  file for more security or leave it as it is.
-   Build the project:  `mvn package [-DskipTests]`

#### Production mode

[](https://github.com/sqshq/piggymetrics/blob/master/README.md#production-mode)

In this mode, all latest images will be pulled from Docker Hub. Just copy  `docker-compose.yml`  and hit  `docker-compose up`

#### Development mode

[](https://github.com/sqshq/piggymetrics/blob/master/README.md#development-mode)

If you'd like to build images yourself, you have to clone the repository and build artifacts using maven. After that, run  `docker-compose -f docker-compose.yml -f docker-compose.dev.yml up`

`docker-compose.dev.yml`  inherits  `docker-compose.yml`  with additional possibility to build images locally and expose all containers ports for convenient development.

If you'd like to start applications in Intellij Idea you need to either use  [EnvFile plugin](https://plugins.jetbrains.com/plugin/7861-envfile)  or manually export environment variables listed in  `.env`  file (make sure they were exported:  `printenv`)

#### Important endpoints

[](https://github.com/sqshq/piggymetrics/blob/master/README.md#important-endpoints)

-   [http://localhost:80](http://localhost/)  - Gateway
-   [http://localhost:8761](http://localhost:8761/)  - Eureka Dashboard
-   [http://localhost:9000/hystrix](http://localhost:9000/hystrix)  - Hystrix Dashboard (Turbine stream link:  `http://turbine-stream-service:8080/turbine/turbine.stream`)
-   [http://localhost:15672](http://localhost:15672/)  - RabbitMq management (default login/password: guest/guest)

## Contributions are welcome!

[](https://github.com/sqshq/piggymetrics/blob/master/README.md#contributions-are-welcome)

PiggyMetrics is open source, and would greatly appreciate your help. Feel free to suggest and implement any improvements.
