---
layout: post
title: "PHPUnit vs PHPSpec: theory on Behaviour Driven Development 1/3"
description: "PHPUnit vs PHPSpec: theory on Behaviour Driven Development 1/3"
category: BDD 
tags: [phpunit, tdd, bdd, php, test, phpspec]
published: true
---
{% include JB/setup %}

The first part of a series of short articles (maybe three) where I'd like to share my tiny experience with BDD and in particular with PHPSpec.

## PHPUnit vs [PHPSpec](https://github.com/phpspec/phpspec2): A word on behaviour testing

I recently discovered that there is an alternative to PHPUnit, and I really like this new approach.

**Q:** But why an alternative if PHPUnit does everything I need?

**A:** What could be changed is the way of doing test,

If you've ever worked in TDD, you know it's very time consuming

1. create - create or modify the test

2. fail - see it fail

3. pass - code the minimum to get the test passed

4. refactor - Refactoring

5. go to step 1



## The differences in a sentence

If the story is a key of the development, the behaviour is the differrence between TDD and BDD.

If you need to test the insertion of an object into a collection and the collection is represented by an Array, with xUnit you should assert that the collection contains the object in the Array, but if the collection will change to another type of container, graph for example, the xUnit will fail, even if the behaviour is unchanged.
In BDD you are not testing you are describing what that classes will do, and so you can continue creating the class (PHPSpec will create the class and the methods for you)


## BDD

There are several BDD frameworks on PHP depending on what you want to test.

#### External behaviour

[Behat](http://behat.org/) deals to have specifications that reflects the environment from the outside.

#### Internal behaviour

[PHPSpec](https://github.com/phpspec/phpspec2) responds to the behaviour in the lower level, from the internal of the classes.

[PHPSpec](https://github.com/phpspec/phpspec2) is considered a tool that helps you to develop.

## Summary

The BDD tests what the object does instead of what it is,
and what it does is much more important.

We have to say that Ruby community helped a lot the evolution of BDD, RSpec is a standard-de-facto, in Ruby world.

## FAQ

**Q:** When I should use PHPUnit and when to use [PHPSpec](https://github.com/phpspec/phpspec2)?

**A:** There isn't a better way between the two, depending on how you want to approach the problem, if you want to follow the behaviour or a unit test the code.

**Q:** With PHPUnit I could do the same things as I could do with [PHPSpec](https://github.com/phpspec/phpspec2)?

**A:** Absolutely, in fact there is a mapping between the two

• Assertion Becomes expectation.

• Test method Becomes code example

• Test case example Becomes group

**BUT** You would be distracted by the code loosing focus on the result, and you might miss the concept of behaviour. \[EDITED 07-01-2012 thanks to [euxpom ![exupom](http://gravatar.com/avatar/77014411a56fa9c746d6a89b7fd96121?size=20)](http://twitter.com/euxpom)\]


## [PHPSpec](https://github.com/phpspec/phpspec2) advantages

xSpec is context specific, expectation, the output is the documentation
and it is for this reason that the language is used because the same syntax you guide you to focus in behaviour, and the importance of documentation.

    bin/phpspec run -f prettify -v

So the big change is on how you write test code.
 

**In the next post I will show some examples on how you could use [PHPSpec](https://github.com/phpspec/phpspec2) to test some (in)famous design pattern.**


## Bibliography

1. [The RSpec Book Behaviour-Driven Development with RSpec, Cucumber, and Friends](http://pragprog.com/book/achbd/the-rspec-book) 
by David Chelimsky, Dave Astels, Zach Dennis, Aslak Hellesøy, Bryan Helmkamp, Dan North

2. Test Driven Development. By Example di Kent Beck (dic. 2002) 

