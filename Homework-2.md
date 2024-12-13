# ECS160-HW2

## Problem: Moderation, and tagging of social media posts

_Learning objectives_
1. Software architecture
   - Pipeline architecture using microservices
   - Microservice architecture
   - Message passing using REST API
3. Libraries and Frameworks:
   - Spring Boot (for Microservices)

_Necessary background knowledge_
1. Java annotations
2. HTTP Get and Post methods. (If this is new to you check out the resource at [w3schools](https://www.w3schools.com/tags/ref_httpmethods.asp)]
3. Basic networking concepts such as ports, etc.

_Problem Statement_

You are provided with an `input.json` file that consists of thousands of social media posts from [Bluesky](www.bsky.app). Just as in HW1 you will parse these social media posts into Java classes. (You can reuse the code you have 
But this time instead of running basic statistical analysis on these posts, you will design a pipeline of microservices that consists of the two microservices described. Each microservice will take the contents of a single message as input and process it,
depending on the functionality of the microservice.

- Microservice 1: A moderation service that checks the contents of the post against a list of "bad words." The moderation service returns `FAILED` if the post content fails the moderation. If it succeeds, it forwards the request to the next microservice. It will return the
results of the second microservice to the client.
- Microservice 2: A tagging service that will analyze the contents of the post and tag the post if it is discussing software security. To perform this check you will match the contents of the post against the list of security keywords. The service will return either `#security` (if the contents of the post match any of the security keywords), or an empty string if it does not. Sample keywords to match against are `[security, encryption,
decryption, Diffie Helman, password, ...]`.
For example, this service could analyze a post `"We should always encrypt passwords"` and return `#security`.

For each post and reply you will send an individual request to the microservice. Make sure to execute the pipeline on both posts _and_ their replies.

**Implementing one microservice**

We will use [Spring Boot](https://spring.io/projects/spring-boot) as our microservices framework. Spring Boot uses Java Annotations to annotate the services. Check out this
[tutorial](https://codecrunch.org/creating-a-post-and-get-request-springboot-ff6e82a5d46b) for using Spring Boot 
with HTTP Post requests. We will create REST API endpoints for these microservices. For more information on REST API check out [this](https://www.redhat.com/en/topics/api/what-is-a-rest-api) link. 

REST APIs can support any HTTP methods including `GET`, `POST`, `PUT`, `DELETE` and so on. In this case we will use the HTTP `POST` method to send the request to the microservice.

A Spring Boot microservice should at least consist of a `SpringBootApplication` and a `Controller`. The Controller specifies the REST endpoint (the url the service is bound to). 

Sample code for a Spring Boot service application is as follows. The class must be annotated with `@SpringBootApplication`. 
```
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ModerationService {

	public static void main(String[] args) {
		SpringApplication.run(ModerationService.class, args);
	}

}
```

Sample code for a Controller is as follows. The controller class must be annotated with the `RestController` annotation. We will be using the `HTTP Post` method to send the request to the microservice, so we annotate the function that implements the REST endpoint
as `@PostMapping("/<endpoint_url>")`. The function itself takes a single argument that maps every parameter of the HTTP Post request into an object of a Java class. This argument must be annotated with the annotation `RequestBody`.
In this sample code, we create a Java class `MyRequest` with a single field `postContent` which
will encapsulate the contents of the post message. The Spring Boot framework will automatically create an instance of the `RequestBody` class and populate it with the request parameters from the HTTP `POST` method.

````
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class ModerationController {

    @PostMapping("/moderate")
    public String moderate(@RequestBody MyRequest request) {
        // Moderation logic
    }

    static class MyRequest {
        private String postContent;

        public String getPostContent() {
            return postContent;
        }

        public void setPostContent(String postContent) {
            this.postContent = postContent;
        }
    }
}
````

Remember, a microservice is an independent piece of software. So, every microservice should be developed an individual IntelliJ project. Every microservice will launch its own Tomcat server, so you need to specify a different port for each microservice. 
Edit the `application.properties` file to add `server.port=3000X` for each microservice. 

Build the application with `mvn clean install`.

Run the application with `mvn spring-boot:run`. 

On a successful launch, you should see log messages similar to the following on the terminal.

````
2024-12-11T20:51:18.604-08:00  INFO 20432 --- [hw2] [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port 30000 (http)
2024-12-11T20:51:18.625-08:00  INFO 20432 --- [hw2] [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2024-12-11T20:51:18.626-08:00  INFO 20432 --- [hw2] [           main] o.apache.catalina.core.StandardEngine    : Starting Servlet engine: [Apache Tomcat/10.1.33]
2024-12-11T20:51:18.751-08:00  INFO 20432 --- [hw2] [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2024-12-11T20:51:18.757-08:00  INFO 20432 --- [hw2] [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 1175 ms
2024-12-11T20:51:19.188-08:00  INFO 20432 --- [hw2] [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port 30000 (http) with context path '/'
2024-12-11T20:51:19.194-08:00  INFO 20432 --- [hw2] [           main] com.ecs160.hw2.ModerationService         : Started ModerationService in 2.082 seconds (process running for 2.372)
````

Once the microservice is up and running, you can test it by querying it using `curl`. 

Syntax for Windows:

```
curl.exe -X POST http://localhost:30000/moderate -H "Content-type: application/json" -d '{\"postContent\": \"Hello, duck Spring Boot!\"}'
```

Syntax for Linux:
```Linux
curl -X POST http://localhost:30000/moderate -H "Content-type: application/json" -d '{\"postContent\": \"Hello, duck Spring Boot!\"}'
```

**Chaining microservices to form a pipeline**

To chain the microservices we need to invoke the second microservice from the first. 

