# ECS160-HW3

## _Due date: 3/10_
## Problem: Moderation, and hashtagging of social media posts

_Learning objectives_
1. Software architecture
   - Pipeline architecture using microservices
   - Microservice architecture
   - Message passing using REST API
3. Libraries and Frameworks:
   - Spring Boot (for Microservices)
   - LLM integration using ollama.

_Necessary background knowledge_
1. Java annotations
2. HTTP Get and Post methods. (See [w3schools](https://www.w3schools.com/tags/ref_httpmethods.asp)]
3. Basic networking concepts such as IP address, ports, etc.

_Problem Statement_

You are provided with an `input.json` file that consists of thousands of social media posts from Bluesky [here](https://github.com/davsec-teaching/ECS160-HW3-skeleton/blob/master/src/main/resources/input.json). Just as in HW1 and HW2 you will parse these social media posts into Java classes. As in HW2, you can ignore replies of replies.
You can reuse the code you already have, but
this time instead of running basic statistical analysis on these posts, you will design a pipeline of microservices that consists of the two microservices described. Each microservice will take the contents of a single message as input and process it,
depending on the functionality of the microservice.

- Microservice 1: A moderation service that checks the contents of the post against a list of "banned words" listed at the end of the asssingment. The moderation service should return `FAILED` if the post content fails the moderation. If it succeeds, it should forward the request to the next microservice and will ultimately return the
results of the second microservice to the client.
- Microservice 2: A hashtagging service that will analyze the contents of the post and tag the post with a hashtag, like `#vacation` and `#happy`. You will invoke a locally running instance of [LLAMA-3](https://www.llama.com/) for the analysis (more later).

You will execute the pipeline on the top-10 most-liked top-level posts in `input.json`. For each of these ten top-level posts, you will send individual requests for both the post and any of its replies to the microservice. In other words, make sure to execute the pipeline on the 10 most-liked posts _and_ their replies.
The output of the pipeline will either by `FAILED` or the hashtag. If the LLM refuses to generate a hashtag for some reason, you can tag it a default tag such as `#bskypost`. In case any post or reply fails the moderation, display it as `[DELETED]`. For other posts and replies, append the hashtag to the post/reply content. For example, if the top-level post and one of its replies fails the moderation, you would display it as follows. 

```
[DELETED]
--> reply content #sample_hashtag1
--> reply content #sample_hashtag2
--> [DELETED] 
```

**Implementing a microservice**

We will use [Spring Boot](https://spring.io/projects/spring-boot) as our microservices framework. Please clone the repository at https://github.com/davsec-teaching/ECS160-HW3-skeleton. This repository comes with
a `pom.xml` file that already contains the library dependencies for Spring Boot. (If you're interested in setting this up yourself, check out https://start.spring.io/). The provided repository contains a `ModerationService.java` source code file that you can use to get started. 

Spring Boot uses Java Annotations to annotate the services. Check out this
[tutorial](https://codecrunch.org/creating-a-post-and-get-request-springboot-ff6e82a5d46b) for how to use Spring Boot 
with HTTP Post requests. 
We will create REST API endpoints for these microservices. For more information on REST API check out [this](https://www.redhat.com/en/topics/api/what-is-a-rest-api) link. 
REST APIs can support any HTTP methods including `GET`, `POST`, `PUT`, `DELETE` and so on. In our case we will use the HTTP `POST` method to send the request to the microservice.

A Spring Boot microservice should at least consist of a `SpringBootApplication` and a `Controller`. The `SpringBootApplication` is a Java class that is annotated with `@SpringBootApplication.` 
This class will implement the `public static void main(..)` function as shown below. 

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

The Controller specifies the REST endpoint (the url the service is bound to). Sample code for a Controller is as follows. The controller class must be annotated with the `RestController` annotation. We will be using the `HTTP Post` method to send the request to the microservice, so we annotate the function that implements the REST endpoint
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
2024-12-11T20:51:18.604-08:00  INFO 20432 --- [HW3] [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port 30000 (http)
2024-12-11T20:51:18.625-08:00  INFO 20432 --- [HW3] [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2024-12-11T20:51:18.626-08:00  INFO 20432 --- [HW3] [           main] o.apache.catalina.core.StandardEngine    : Starting Servlet engine: [Apache Tomcat/10.1.33]
2024-12-11T20:51:18.751-08:00  INFO 20432 --- [HW3] [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2024-12-11T20:51:18.757-08:00  INFO 20432 --- [HW3] [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 1175 ms
2024-12-11T20:51:19.188-08:00  INFO 20432 --- [HW3] [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port 30000 (http) with context path '/'
2024-12-11T20:51:19.194-08:00  INFO 20432 --- [HW3] [           main] com.ecs160.HW3.ModerationService         : Started ModerationService in 2.082 seconds (process running for 2.372)
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

Develop the second microservice the same way as the first. Create a separate project for it and assign it a different port. The moderation microservice should invoke the hashtagging microservice only if the moderation passes.

To chain the microservices we need to invoke the second microservice from the first. We can use the `HttpClient` provided by the Java standard libraries to invoke the second microservice from the first. Sample code
for that is as follows. Remember that we need to send a Json request formatted as `{"postContent": "This is a sample post"}` over HTTP Post. Check out the tutorial [here](https://openjdk.org/groups/net/httpclient/intro.html) for documentation on how to use `HttpClient`. You are welcome to use any other library or RPC library. 
As always add JUnit test cases for your microservices and verify that they pass. The unit tests for the moderation microservice should **not** assume that the second microservice is up and running. Similarly, the unit test for the tagging microservice should **not** assume that the Ollama instance is up and running.

Invoke the client using `curl` as shown above. This time it might print `#security` if the post contains a security-related keyword. And finally, write a third Java application that will invoke this microservice
pipeline for each of the most-liked posts in `input.json`.

**LLM integration**

Local Language Models are perhaps the most exciting (or at least most hyped) thing happening currently in Computer Science right now. Meta has released their LLAMA foundational model as open source. For this assignment, we will first 
run the LLAMA-3 model locally. Then we will query this model and ask it to hashtag our top-10 most-liked posts from `input.json`. 

First install ollama from [here](https://ollama.com/download). 
Then, follow the instructions [here](https://github.com/ollama/ollama?tab=readme-ov-file#quickstart) to download the LLAMA-3 model on your local machine. 

We will use the [Ollama4j](https://github.com/ollama4j/ollama4j) library to invoke the LLM from our Hashtagging microservice. The instructions for how to add the Maven dependency and how to invoke the LLM from Java code is available in the library's documentation. Make sure that the prompt you send has a directive such as `Please generate a hashtag for this social media post` followed by the post itself.

**List of banned words**
1. illegal
2. fraud
3. scam
4. exploit
5. dox
6. swatting
7. hack
8. crypto
9. bots
     
**Submission instructions**

Your final submission will consist of three projects - two microservices and one main application that invokes these microservices. Please zip these projects together into a single Zip file and upload on Canvas. 
