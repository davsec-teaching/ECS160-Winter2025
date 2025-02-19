# ECS160-HW2
## _(Due date: 2/19)_
## Problem: Persistence framework using Java Annotations and Reflection 

_Learning objectives:_ 
1. Java annotations
2. Java reflection
3. Dynamic proxies using Javassist (Extra Credit)

_**(Total points: 20 + 10 (Extra credit))**_

_Problem Statement:_

Just as in the previous assignment, you are provided with an `input.json` file located [here](https://github.com/davsec-teaching/ECS160-HW2-skeleton/blob/master/src/main/resources/input.json) that consists of thousands of social media posts from [Bluesky](https://bsky.app). Each post can have one or more replies. **_For this assignment you can ignore replies-to-replies._**

Just as in the previous assignment, you will parse the JSON file and create Java objects from the JSON content. In this assignment you will ensure that your Post class has the following fields - 
1. `postId` (of type Integer)
2. `postContent` (of type String)
3. `replies` (of type List<Post>)

If your Post class from the previous assignment does not have this structure, please modify it to use this structure. Also, this assignment assumes that you are keeping a reference from the post to its replies in the Redis data-store in the follow way - you are appending all the reply Ids into a comma-separated string and store them with a `replyIds` key. If your code doesn't currently do this, please modify it to do this.

This assignment's goal is to develop a Redis persistence framework that can persist and load objects of _any_ type, and test it with the posts and replies from the JSON file. Your persistence framework will be in a separate Java pacakge named `com.ecs160.persistence`. All annotations and the `Session` class (described below) should be inside this package.

**Annotations**
This persistence framework will provide the following annotations - 

1. `@Persistable` - If a programmer marks any class with this annotation it should be saveable in a Redis database. 
2. `@PersistableField` - For every class annotated with `@Persistable`, only the fields annotated with `@PersistableField` are saved in the Redis database. Note that the field _must_ be private. You will lose points, if you change the field to `public`.
3. `@PersistableId` - Denotes that this field is the ID of the class. This will also act as the "key" in Redis's data-store.
4. `@PersistableListField` - This annotation represents a field which is a `List`. This annotation should support a field `className`. This field will contain the fully qualified class name of the list element object types. 
5. [**_Extra credit_**] `@LazyLoad` - If supporting lazy loading for extra credit, this annotation will indicate that the field is lazy loaded.

For example:

```
@Persistable
public class MyOtherClass {
   // ... 
}
@Persistable
public class MyClass {
   @PersistableId
   private Integer id; // this field will be persisted and is the ID
   @PersistableField
   private Integer number; // this field will be persisted
   
   private String msg; // this field will not be persisted

   @PersistableListField(className="MyOtherClass")
   private List<MyOtherClass> listOfItems; // this will be persisted.
                                             // @PersistableListField indicates this is a list
                                             // and the className field indicates that the
                                             // list elements are of type MyOtherClass
}
```

**Important:** - No class/annotation/interface in the `com.ecs160.persistence` package should contain any logic that is specific to the `Post` type. Similarly, no logic related to persistence should be in the the `Post` type. All accesses and updates must be performed using reflection and/or proxies.

**Session class**

You should create a class `Session` in the `com.ecs160.persistence` package. This class will contain the Redis persistence code (using Jedis). This class should support the following two functions - 
1. `void add(Object obj)` - This method should add an object to the `Session`, but not persist it until `persistAll()` is invoked.
2. `void persistAll()` - This method will persist all the objects added to the `Session`. During this operation, if a field is annotated with `@PersistableListField`, for each list item, the corresponding field annotated as `@PersistableId` should be extracted and appended to a comma-separated string, and then stored in Redis by the persistence framework.
3. `Object load(Object object)` - This method will accept an Object with only the field annotated `@PersistableId` populated and load the entire object from the Redis database (the field annotated with `@PersistableId` is the key in the Redis data-store). In the case of fields annotated as `@PersistableListField`, these fields will be stored in a comma-separated string form in the Redis data-store, as described above.
4. **Extra credit (10 points)** - The  `load()` method should lazily load the fields marked with the annotation `@LazyLoad`. Please check the instructions under "Lazy Loading" for more information.

**Assumptions**
You can make the following assumptions - 
1. Replies to replies are ignored.
2. Only fields of type `Integer` and `String` are persistable.
3. Fields annotated `@PersistableListField` do not store primitive types.
4. If a field is annotated as `@PersistableListField` the field type is indeed `java.util.List`.
5. Extra credit: fields annotated as `@LazyLoad` is always of type `@PersistableListField.`
6. Extra credit: the getters of fields annotated as `@LazyLoad` always follow the [Java Bean Naming convention](https://www.oreilly.com/library/view/javaserver-pages-3rd/0596005636/ch20s01s01.html)


   
**Main class functionality**

The following is the main class functionality. 

1. The main class should load the `input.json` file and create objects of type `Post`. 
2. It should create an object of type `Session` and add all the `Post` objects to this `Session` object.
3. Then, it should call `persistAll` on the `Session` to persist all the post objects in the database.
4. It should then prompt the user for a post id and read the response from the command line.
5. After reading the post id, it should load it from the Redis data-store using the `load` method in the `Session` object.
6. It should then print the post content and its replies on the command line in this way -

```
> [Post message]
> --> [Reply message]
> --> [Reply message]
> --> [Reply message]
```

The exact formatting isn't important, but it should be easy to determine which is the post and which are the replies. 

7. **For extra credit: ** Because the `replies` field in `Post` is marked as `Lazy`, `load` should only load the post message and not the replies. The program should ask the user if it should continue to load the replies. If the programmer presses any key, it should load the replies and print them on the command line. 

**Extra Credit (10 points): Lazy loading with `@LazyLoad`**

To support lazy-loading, you should first create an annotation `@LazyLoad`. You should then annotated the `replies` field in the `Post` class with this annotation. The `Session` class' `load` method should create 
`partially populated` objects for all items of the list type , where only the the field annotated as `@PersistableId` is populated. Then, when the `getter` method for that field is invoked, the `Session` class should 
hit the Redis data-store and fully populate all the objects in the list type. 

For example, in the case of the `Post` class, where the `replies` field is annotated as `LazyLoad`, executing `session.load(post)` will return a `Post` object with partially populated reply objects where only the `postId` field is populated. When `getReplies` is invoked, then the `Session` object will fetch and populate the reply objects completely.

Hints - 
1. You want to create and return a dynamic proxy when loading an object that contains a `@LazyLoad` annotation. You can create a `Javassist` proxy as discussed in class (refer to slide), for this purpose.
2. You can completely ignore performance for this assignment.
   
**Grading**

- 14 points: Successfully saving and loading top-level posts to/from the Redis data-store using Reflection and annotations. Note that to get these 14 points, you do not need to support the `@PersistableListField` annotation, or even parse the replies of the threads from the JSON file.
- 6 points: Successfully saving and loading top-level posts and replies to/from the Redis data-store using Reflection and annotations.
- 10 points (Extra credit): Successfully saving and loading top-level posts and lazy-loading replies to/from the Redis data-store using Reflection and annotations.


**Getting started**

You will follow the same steps to setup your project by cloning the skeleton repo [here](https://github.com/davsec-teaching/ECS160-HW2-skeleton). Add all the dependencies in `pom.xml` from the previous assignment that you think will be useful in this assignment. 
Make sure your Redis server is listening to port `6379`. If implementing the lazy loading for extra credit, please also add the Javassist library to pom.xml.


**_Submission_**

Please commit your code to your Github repo. For your submission, please download and zip your repository and upload on the Canvas link. 

