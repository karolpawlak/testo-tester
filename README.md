# Testo Tester
A troubleshooting swiss-army knife application

---

## Installation
The microservices in this project are written in Java with Spring Boot. If you download the project, each microservice can be ran locally in the IDE or with the following command:
```
./mvnw clean spring-boot:run
```
In order to deploy the app in the cloud, the code must be compiled and packaged to a JAR file using Maven wrapper. To skip tests during the packaging, add `-DskipTests` flag to compile the tests without running them or `-Dmaven.test.skip` to not compile the tests at all.:
```
./mvnw clean package
```

You can also compile and package the code, and also put it in your local repository so that other projects can refer to it. This can be done using the following command which will place all the needed components (dependencies) in a directory called **.m2** uder the user's folder.
```
./mvnw clean install
```

This will add the JAR file to the **target** folder, which can then be used to create a container or directly push it to Cloud Foundry. The `clean` option will delete the previously created **target** folder. The applications that have their Maven plugin configured in their pom file, have layers enabled and as a result, enforcing the creation of a layered JAR file. With layered JARs, the structure looks similar to a typical Spring Boot fat JAR which is composed of 3 main areas - bootstrap classes required to launch the Spring app, application code and 3rd party libraries. However, this time we get a new layers.idx file that maps each directory in the fat JAR to a layer.

The goal is to place application code and third-party libraries into layers that reflect how often they change. With this configuration, the Maven package command (along with any of its dependent commands) will generate a new layered JAR using the four default layers

- "dependencies":
  - "BOOT-INF/lib/"
- "spring-boot-loader":
  - "org/"
- "snapshot-dependencies":
- "application":
  - "BOOT-INF/classes/"
  - "BOOT-INF/classpath.idx"
  - "BOOT-INF/layers.idx"
  - "META-INF/"

This can be observed when examining the JAR file itself and running the command from within the **target** folder `java -Djarmode=layertools -jar testotester-server-0.0.1-SNAPSHOT.jar list`


More information on Maven's application build lifecycle [here](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html).

Note. If you have Maven CLI installed on your local machine, the `./mvnw` can be substituted for `mvn`.

## Creating an image
Each application has multiple versions of the Dockerfiles which are used to create images. The **Dockerfile** has basic configuration for an image with a `WORKDIR` and `VOLUME`. The **Dockerfile.Layered** extracts the layers from our fat JAR, then copies each layer into the Docker image. Each COPY directive results in a new layer in the final Docker image thus creating a more tailored Docker image.

## Cloud Foundry deployment
Prerequisites:
* [Command-Line Interface for Cloud Foundry](http://docs.cloudfoundry.org/cf-cli/)

Using the `cf push` command, each microservice can be pushed using settings in the provided `manifest.yml` file. The output from the command will show the URL that has been assigned to the application.

## Actuator
Each microservice contains an Actuator dependency that enables production-grade tools for exposing operational information about the running application such as monitoring the app, gathering metrics, understanding traffic and examining the status of database connectivity. It uses HTTP endpoint interact with it.

[More information about the Actuator API can be found in the official documentation](https://docs.spring.io/spring-boot/docs/current/actuator-api/htmlsingle/). Spring Boot enables a discovery endpoint `/actuator/` that returns all available Actuator endpoints as hypermedia. 

The URLs are in a form of `/actuator/{id}` where `id` is one of the endpoints listed below eg. `/actuator/info`
List of all available Actuator endpoints:
* `/auditevents` lists security audit-related events. Requires an *AuditEventRepository* bean.
* `/beans` returns all available Spring beans in our *BeanFactory*.
* `/caches` returns available caches.
* `/conditions` builds a report of conditions that were evaluated on configuration and auto-configuration classes.
* `/configprops` allows us to fetch all *@ConfigurationProperties* beans.
* `/env` returns the current environment properties from Spring's *ConfigurableEnvironment*. Additionally, we can retrieve single properties.
* `/flyway` provides details about our *Flyway* database migrations. Requires one or more *Flyway* beans.
* `/health` shows application health information.
* `/health/disk` shows application disk usage information.
* `/heapdump` builds and returns a heap dump from the JVM used by the application.
* `/httptrace` displays HTTP trace information (by default, the last 100 HTTP request-response exchanges). Requires an *HttpTraceRepository* bean.
* `/info` returns general information.
* `/integrationgraph` shows the Spring Integration graph. Requires a dependency on *spring-integration-core*.
* `/liquibase` provides details about our *Liquibase* database migrations. Requires one or more *Liquibase* beans.
* `/loggers` shows and modifies the configuration of loggers in the application.
* `/metrics` shows which application metrics information are available.
* `/mappings` displays a collated list of all *@RequestMapping* paths.
* `/quartz` shows information about Quartz Scheduler jobs.
* `/scheduledtasks` provides details about every scheduled task within the application.
* `/sessions allows` retrieval and deletion of user sessions from a Spring Session-backed session store. Requires a servlet-based web application that uses *Spring Session*.
* `/shutdown` performs a graceful shutdown of the application. Disabled by default.
* `/startup` shows the startup steps data collected by the *ApplicationStartup*. Requires the *@SpringApplication* to be configured with a *BufferingApplicationStartup*.
* `/threaddump` dumps the thread information of the underlying JVM.

## Eureka Server

Eureka server enables service registry to allow client-side service discovery between the microservices. This allows the services to find and communicate with each other without hard-coding the hostname and port. Eureka server provides a fixed point in this architecture as a service registry, with which each service has to register. The Eureka server provides a dashboard available at `http://localhost:8761` that allows to inspect the registered service instances.

## Web

Specific information regarding the Web microservice.

## Server

Specific information regarding the Server microservice.

### Selecting a profile
A Spring profile should be activated to choose the database provider that the application should use. The profile is selected by setting the system property `spring.profiles.active` when starting the app. By default, the application will start with `in-memory` profile. 

This property can be set programatically in the `application.properties` file or specified in Cloud Foundry deployment manifest `manifest.yml` in the format `spring.profiles.active=<PROFILE>` where `<PROFILE>` is one of the following available profiles:

* `in-memory` (no external database required)
* `mysql`
* `postgres`
* `redis`

## Client

Specific information regarding the Client microservice.