# ECS160-HW4 (Extra credit - 10 points)

## _Due date: 3/21_
## Problem: Redesign and extend HW1 using design patterns

_Learning objectives_
1. Design patterns
   - Creational patterns
   - Structural patterns
   - Behavioral patterns
  
_Problem statement_

This homework assignment is different from the previous three assignments.
In this homework assignment, you will redesign HW1 using your knowledge of design patterns. In addition to submitting the code, you will also submit a document explaining your choice of design pattern.

### Requirements
The following are the requirements. You can reuse the skeleton code from HW1 and HW3, and like in the previous
homeworks, you can use the `input.json` file provided as input. You can ignore _all_ replies and only consider the top-level posts and their immediate replies for this assignment (no need to consider reply-of-replies).

#### Redesign HW1 
1. Ensure that the application can accept two configuration options---1) analysis type (weighted, non-weighted), and 2) the name of the JSON file. Store these configuration options in a class. Using a design pattern
ensure that this configuration object is available readily in the analysis classes, and also ensure that _only_ a single object of this configuration class can exist during the application's lifetime. _(Which design pattern
should you use?)_

2. Use a design pattern to ensure that both single posts (with no replies) and threads can be handled using the same API. Modify any logic where you handle posts and threads differently. _(Which design pattern should you
use?)_

3. Use design patterns to compute the statistics. These design pattern classes should visit each post and reply for all top-level posts and threads in `input.json` and compute the required statistic. _(Which design
pattern should you use? How many classes did you create?_)

#### Extend HW1

1. Extend HW1 to add the hashtagging functionality from HW3. In other words, we will again run the LLAMA-3 model locally, using Ollama, and then
query this model and ask it to hashtag our top-10 most-liked posts from input.json. For this assignment, you should store this hashtag in a post/thread object, but you should **not** neither add a `hashtag` field
to the class, nor subclass the `Post` or `Thread` class. _(Which design pattern should you use?)_ You should then write a method `printHashTag()` that prints the hashtags for all posts and threads, including those
that donot have a hashtag.

You do not have to develop a microservice for this homework.

**_Submission_**
You will submit the code, and a document outlining the design pattern you chose for each of these cases and why you chose them. 

