# ECS160-HW2
## _(Due date: 2/14)_
## Problem: Persistence framework using Java Annotations and Reflection 

_Learning objectives:_ 
1. Java annotations and reflection

   
_Problem Statement:_

Just as in the previous assignment, you are provided with an `input.json` file located [here](https://github.com/davsec-teaching/ECS160-HW2-skeleton/blob/master/src/main/resources/input.json) that consists of thousands of social media posts from [Bluesky](https://bsky.app). Every post can contain one of more replies. If a post does not have any reply we will call it a `top-level
post`.

In the previous assignment, you parsed the Json file and created Java objects from the data provided. In this assignment your goal is to persist the Java objects corresponding to top-level posts in a Redis database.
To ensure that your 
application's data model is completely decoupled from the persistence logic, you will create an independent persistence layer that can handle objects of _**any**_ class, as long as the class is annotated with a 
`@Persistable` annotation. (See below for the details of the Annotations you must support.)



**Getting started**

Create a new IntelliJ project and sync it with a new Github repo. Add all the dependencies in `pom.xml` from the previous
assignment that you think will be useful in this assignment. 

We will persist the social media posts in a Redis database. [Redis](https://redis.io/) is a simple key-value store, which supports persistence. If you don't have redis installed, install it on your local machine by following the instructions 
[here](https://redis.io/docs/latest/operate/oss_and_stack/install/install-redis/). Make sure your Redis server is listening to port `6379`. We will also need the [Jedis](https://github.com/redis/jedis) library to simplify the communication with the Redis server. So add the dependency for Jedis to `pom.xml`. 
Any version of Jedis `> 4.3.1` should suffice. Then run `mvn clean install` to ensure that the jar files are fetched.

**Annotations and API**

Your persistence framework support the following annotations:
1. `@Persistable` If a programmer marks any class with this annotation it should be saveable in a Redis database. 
2. `@PersistableField` For every class annotated with `@Persistable`, only the fields annotated with `@PersistableField` are saved in the Redis database.

For example:

```
@Persistable
public class MyClass {
   @PersistableField
   private int id; // this field will be saved
   @PersistableField
   private int number; // this field will be saved
   
   private String msg; // this field will not be saved
}
```
Note that by default if you try to `get` the value of a `private` field using Reflection, it will throw an `IllegalFieldAccessException`. You should first set the field as accessible using `persistable_field.setAccessible(true)`. **DO NOT** change the visibility of the field in the class declaration. You will lose points if you do that. 

You should also support the following API-
1. `RedisSession redisSession = new RedisSession(url, port)`
2. `redisSession.add(obj)` Here `obj` is any object of a type that is annotated with `@Persistable`.
3. `redisSession.persistAll()` This will convert (or serialize) the object to key-value pairs and store them in the Redis database.

**Object serialization and persistence**
You can use any strategy for serializing the social media post into a key-value pair. Note that however, the key must be unique. So good candidates for the key are a alphanumeric identifier you assign to each post, such
as `post001`, `post002`, and so on. The `value` should be a comma-separated concatenation of all post attributes, such as like count, reply count, etc., in addition to the post content.

You can use the following code as the starting point for the communication with Redis using Jedis

```
      String key = ... ;
      String value = ...; 
      Jedis jedisSession = new Jedis("localhost", 6379);
      jedisSession.set(key, value);
```

Redis comes with a command-line tool that you can use to test that the objects were indeed persisted in Redis database. Run the following command and it should dump out all the key-value pairs persisted.
```
redis-cli --scan | while read key; do echo "$key -> $(redis-cli GET "$key")"; done
```

As always, create JUnit test cases to test the persistence framework. You can use Jedis in the JUnit test-case to verify that the object got persisted.

**_Submission_**

Please commit your code to the Github repo and tag it. Your submission should contain a single link to the repository and the tag. 

