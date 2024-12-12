# ECS160-HW1

## Problem 1: Basic analysis of social media posts 

_Learning objectives:_ 
1. Java basics: Encapsulation, Inheritance, File I/O, Exceptions.
2. Design pattern: Recursion, Composite design pattern, Singleton design pattern.
3. Testing: JUnit, mock-testing, Github integration via Github Actions.
4. Tools and libraries: Maven, adding dependencies to `pom.xml`, Gson for parsing JSON files, Apache's Common CLI for parsing command line interfaces.

_Problem Statement:_

You are provided with an `input.json` file that consists of thousands of social media posts from [Bluesky](www.bsky.app). Your goal is to write a Java program that computes certain basic statistics for the provided posts. These statistics are---the total number of posts, the average number of comments per post, and average interval between comments (for posts which have comments). Depending on an option provided on the command line (`weighted = true|false`), you will either compute a simple average, or a weighted average (TODO: specify formula) that depends on the length of the post or comments. The full path of the `input.json` file will also be provided on the command line.

**Adding library dependencies.**

First, add the dependencies for Gson and commons-cli to `pom.xml`. Read the [Maven tutorial](https://maven.apache.org/guides/getting-started/maven-in-five-minutes.html). Because we are using the IntelliJ IDE, if we click on the `Run` button, the Maven build steps will be automatically performed. 

````
    <dependencies>
        <dependency>
            <groupId>com.google.code.gson</groupId>
            <artifactId>gson</artifactId>
            <version>2.10.1</version>
        </dependency>
        <dependency>
            <groupId>commons-cli</groupId>
            <artifactId>commons-cli</artifactId>
            <version>1.5.0</version> <!-- Use the latest version -->
        </dependency>

    </dependencies>
````

**Class design**
Study the `input.json` file provided. JSON (Javascript Object Notation) is a popular data format for exchanging data between applications. Each JsonObject consists of multiple key-value pairs. Read about [JSON](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Objects/JSON) here.

Every social media post can either be a leaf post or consist of child posts, depending on whether or not the `thread` object contains a `replies` key or not. The structure of the JSON document is as follows:
````
{
    "feed": [
        {
            "thread": {
                "$type": "app.bsky.feed.defs#threadViewPost",
                "post": { ... // post contents },
                "replies" : {
                      "post": { .... // reply post contents },
                       ... // other stuff
                      }
                }
         },
         {
           "thread": { ... // thread contents as above }
         }
         ....
      ]
}               
````
The structure describes a nested structure for each thread, where each post can have multiple replies.
1. The `feed` key has a `JSonArray` consisting of multiple `JsonObjects` mapping to the key `thread`.
2. The JSONObject mapping to the `thread` key consists of many key-value pairs. The ones we are interested in are `post` and `replies`.
3. `post` maps to a JsonObject for the post.
4. `replies` maps to a JsonArray, where each `reply` JSonObject consists of a key-value pair for `post`.

Use the concepts of Encapsulation, Inheritance, and the _Composite design pattern_ to model the structure described above. Note that you can model this in many different correct ways. To design the classes, you should ask yourself which key-value pairs from the Json you'll need to compute the desired statistics and create fields for these keys in the class. You may or may not choose to use Inheritance, but you should likely use the Composite design pattern to model the relationship where a `Thread` can contain many `Replies`.

**JSON parsing and Statistical Analysis**

We will use (Gson)[https://github.com/google/gson] to parse the `input.json` file. Here is some code to get you started.

```
        URL resource = JsonDeserializer.class.getClassLoader().getResource("input.json");

        JsonParser parser = new JsonParser();

        JsonElement element = parser.parse(new FileReader(resource.toURI().getPath()));

        if (element.isJsonObject()) {
            JsonObject jsonObject = element.getAsJsonObject();

            JsonArray feedArray = jsonObject.get("feed").getAsJsonArray();
            for (JsonElement feedObject: feedArray) {
                // Check if you have the thread key 
                if (feedObject.getAsJsonObject().has("thread")) {
                    // parse the post and any replies (recursively)? 
                } 
            }
        }
```

While parsing the Json file, load the Json objects into the Java classes for the _Composite_ pattern designed earlier.
Create a `Analyzer` interface class with two concrete sub-classes for the non-weighted and weighted calculations. Invoke the right analysis depending on the configuration option provided.


**Testing**
We will write JUnit test cases for the Analysis classes designed in the previous step. Read more about JUnits [here](https://junit.org/junit5/docs/current/user-guide/).

To use JUnit testing framework, we first will have to add the JUnit jar library to `pom.xml` and run `mvn clean install` in IntelliJ terminal.

````
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-api</artifactId>
    <version>5.9.3</version> 
    <scope>test</scope>
</dependency>
````

We will create a new class under the `tests/` directory. If your basic analyzer is called `BasicAnalyzer.java`, then the convention is that your test class should be called `BasicAnalyzerTest.java`. In that class add separate JUnit test methods to test each of the functions of the BasicAnalyzer.java class. Do the same for the weighted analysis class. Make sure to test all corner cases, such as empty posts, empty thread, _very_ long posts, and so on. We will run our own set of JUnit tests on the code, so it is in your benefit if you thoroughly test your code. Add all the required assertion checks.

We will set up a Continuous Integration pipeline where everytime we push, the Maven actions to build and test the application are carried out. Check out [this](https://docs.github.com/en/actions/writing-workflows/quickstart) for basics of Github actions and [Github Actions: Maven](https://docs.github.com/en/actions/use-cases-and-examples/building-and-testing/building-and-testing-java-with-maven) for more Maven specific information.

We will add a badge indicating the status of our build to our project page. First, create a `README.md` at the top-level of the repository. Then, click on the `Actions` tab on the main project repository. Then, click on the name of the action on the left panel. In the right part of the screen, you'll see the ellipsis (...). Click on `Create status badge`. This will give you the Markdown text that you can add to the `README.md`.

**_Submission_**

Please commit your code to the Github repo and tag it. In a text file specify the following:
1. Link to your github and tag. This link should show the "tests passed" logo.
2. Command to clone your repository and apply the tag.
3. Results of the analysis.


Your submission will be run against different (and larger) JSON files.

