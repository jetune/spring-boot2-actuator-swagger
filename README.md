# Spring Boot 2 with Actuator and Swagger

This project is a basic example of a Spring Boot 2 application with the
'loggers' Actuator endpoint enabled and a generated Swagger 2 API page.

The goal of this project is to showcase the fact that currently,
Swagger 2 is unable to generate a correct API doc based on the Actuator endpoints
created by Spring Boot since version 2.

## Usage

Run the application by executing the main method in the [Application class](src/main/java/com/rlippolis/spring/springboot2actuatorswagger/SpringBoot2ActuatorSwaggerApplication.java).
Then, navigate to `http://localhost:8080/swagger-ui.html` to see the generated API doc.

### Error 1: unable to execute GET requests

You will notice that Swagger is unable to execute the GET request to `/actuator/loggers/` (located under `operation-handler`).
When attempting to execute that request, the following error appears:

```TypeError: Failed to execute 'fetch' on 'Window': Request with GET/HEAD method cannot have body.```

This is caused by the fact that Swagger wants to include a request body in the GET request,
which of course is not correct. Spring Actuator maps all endpoints to one method,
`org.springframework.boot.actuate.endpoint.web.servlet.AbstractWebMvcEndpointHandlerMapping.OperationHandler.handle`,
which has an `@RequestBody` annotated method parameter.

Note that the Reactive variant (`org.springframework.boot.actuate.endpoint.web.reactive.AbstractWebFluxEndpointHandlerMapping`),
DOES have a separate `ReadOperationHandler` and `WriteOperationHandler`, which would prevent this problem.

### Error 2: path parameters not recognized

When attempting to execute the `/actuator/loggers/{name}` GET request, you will notice that
Swagger does not allow you to specify the name parameter.

Normally, Swagger recognizes path parameters by the `@PathParam` annotation on method parameters.
However, because of the mapping to the `org.springframework.boot.actuate.endpoint.web.servlet.AbstractWebMvcEndpointHandlerMapping.OperationHandler.handle`
method (which does not have any `@PathParam` annotated method parameters), no path parameters are detected.

### Mitigation

When downgrading to Spring Boot _1.5.10.RELEASE_, the Swagger page will work correctly again. Of course, this is not what we want!

I have created a somewhat dirty fix (see [DirtyFixConfig.java](src/main/java/com/rlippolis/spring/springboot2actuatorswagger/config/DirtyFixConfig.java)) to correct the actuator endpoints.
To enable this fix, set the `dirty.fix.enabled` property to `true` in the `application.properties`.

This fix uses a Swagger plugin to fix the above issues. Of course, I hope this can be fixed in the Spring Boot framework or Swagger library!
