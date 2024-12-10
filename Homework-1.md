# ECS160-HW1

## Problem 1: Basic analysis of social media posts 

_Learning objectives:_ 
1. Java basics: Encapsulation, Inheritance, File I/O, Exceptions.
2. Design pattern: Composite design pattern, Singleton design pattern.
3. Testing: JUnit, mock-testing, Github integration.

_Problem Statement:_

You are provided with an `input.json` file that consists ~10,000 social media posts. Your goal is to write a Java program that computes the average number of comments per post, average interval between comments (for posts which have comments), the average number of posts and comments per user. Depending on an option provided on the command line (`weighted = true|false`), you will either compute a simple average, or a weighted average that depends on the length of the post or comments.


First, spawn a MongoDB instance, create a database `socialmedia`, and store the `input.json` to the database. This step should happen only once, _only if_ the database doesn't already exist and isn't populated.

Create a `Analyzer` interface class with two concrete sub-classes for the non-weighted and weighted calculations. Create a _Singleton_ class to store the configuration option(s) passed to the command line. In this homework, it will pass a single command line option `weighted=true|false`. 

Every social media post can either be a leaf post or consist of child posts. Use the _Composite design pattern_ to model this. 

Load the JSON items into the Java classes for the _Composite_ pattern designed earlier and invoke the weighted or non-weighted analysis.

Write JUnit test cases for these analyses. Create Github Actions so that these test-cases are executed when anyone pushes to this branch. Add a [badge](https://docs.github.com/en/actions/monitoring-and-troubleshooting-workflows/monitoring-workflows/adding-a-workflow-status-badge) to your Repository's README.md that shows the status of the build tests.

_Submission_

Please commit your code to the Github repo and tag it. In a text file specify the following:
1. Link to your github and tag. This link should show the "tests passed" logo.
2. Command to clone your repository and apply the tag.
3. Results of the analysis.


Your submission will be run against different (and larger) JSON files.

