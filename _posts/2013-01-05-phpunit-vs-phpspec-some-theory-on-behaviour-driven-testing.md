---
layout: post
title: "PHPUnit vs PHPSpec: some theory on Behaviour Driven Testing."
description: "PHPUnit vs PHPSpec: some theory on Behaviour Driven Testing."
category: 
tags: []
published: false
---
{% include JB/setup %}

## PHPUnit vs PHPSpec: A word on behavior testing

I recently discovered that there is an alternative to PHPUnit.

**Q:** But what's the alternative if a PHPUnit does everything it needs to do?

**A:** What could be improved is the way to do testing.


If you've ever worked in TDD, you know it's very expensive

1. create - create or modify the test

2. fail - see it fail

3. pass - code the minimum to get the test passed

4. refactor - Refactoring

5. go to step 1

and BDD?

### todo here

dire che il bdd è differente, 
the BDD recommended a different way of doing test, oriented driven behavior that a class should have and not what it should do.


## The difference in a sentence

If the story is a key of the development, the behavior is the differrence between TDD and BDD.

If you need to test the addition of an object in a collection and this is represented by an array, with xUnit you should test that the collection contains in that array the object, but if the collection is changed to another type of container, graph for example the xUnit will fail, even if the behavior is unchanged.


## BDD

There are several BDD frameworks on PHP depending on what you want to test.

#### External behavior

Behat deals to have specifications that reflects the environment from the outside.

#### Internal behaviour

PHPSpec responds to the behavior in the lower level, from the internal of the classes.

PHPSpec is considered a tool that helps you to develop.

## Summary

So the BDD tests what the object to do instead of what it is,
and what it does is much more important.

## FAQ

**Q:** When I should use PHPUnit and when to use PHPSpec?

**A:** There isn't a better way between the two, depending on how you want to approach the problem, if you want to follow the behavior or a unit test the code.


**Q:** With PHPUnit I could do the same things you write about PHPSpec

**A:** Absolutely, in fact there is a mapping between the two

• Assertion Becomes expectation.

• Test method Becomes code example

• Test case example Becomes group

**BUT** linguistically you would be more look at the code that the result, so you might miss the concept of behavior.

## PHPSpec advantages

xSpec is context specific, expectation, the output is the documentation
and it is for this reason that the language is used because the same syntax you guide you to focus in behavior, and the importance of documentation.

`bin/phpspec run -f prettify -v`

