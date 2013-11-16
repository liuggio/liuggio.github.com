---
layout: post
title: "Design by contract behaviour"
description: ""
category: blah-blah-blah
tags: [php-internal, php, interface]
published: true
---
{% include JB/setup %}

In recent years, [Composer](http://getcomposer.org) has given a breath of freshness to the dependencies, and framework such as [Symfony2](http://www.symfony.com)
 have made best practices and decoupling their pride.

The union of these two phenomena have moved the attentions to the type hinting in order to have contracts with `Interfaces`.

PHP doesn't have the support for [Structural type system](http://en.wikipedia.org/wiki/Structural_type_system)

## The problem

The problem comes when a library uses a functionality of a third library, but do not want to hard-code the third-part library in the namespace


## Case Study

I have a library that uses a logger:

    // src/Service/MyService
    namespace MyLib\Service;

    class MyService

       private $log;

       public function __construct($log)
       {
         this->log = $log;
       }

       public function doSomethig()
       {
          $this->log->info('log this');
       }


I want to use the type hinting because that dependency will be called with `info`.

Then I add the type-hinting LoggerInterface, but which namespace to use?
I can't define a new mine LoggerInterface, so use the Monolog one.

    // src/Service/MyService
    namespace MyLib\Service;

    use Monolog\LoggerInterface;

    class MyService

       private $log;

       public function __construct(LoggerInterface $log)
       {
         this->log = $log;
       }

       public function doSomethig()
       {
          $this->log->info('log this');
       }


Doing this I have hardcoded a **dependency** to the Monolog library, composer.json and this file `src/Service/MyService` has to be maintained together,
isn't smell to you?


And what happen when I'll find a better logger library that has got the same LoggerInterface and the function 'info'?
I've to change the namespace to the new LoggerInterface or the type hinting would fail,
 because the control for the type hinting is also on the namespace of the interface and not on the behaviour.


## Solution?

Monolog have fixed this problem by putting a shared interface inside PSR [php-fig.org/](http://www.php-fig.org) repository, but not all cases can be put in the php-fig.

One solution would be to permit the alias for namespaces.

Another `naive` solution would be modify the `php-internal` of how type hinting works,
maybe defining an interface with a sort of keyword eg.`behaviour` that forces the engine to check that
the type that you pass has the same behavior (functions) and not forcing the namespace.


    namespace MyLib;
    Behaviour Interface LoggerInterface
    {
        public function info($string);
    }


## Why?

Can you imagine a world where the libraries are decoupled but they respects the interfaces without sharing file but sharing `behaviour`?

## PHP-fig

PHP-fig is a great tool but it's democratic and democracy on internet is not always able to satisfy the minorities, and to be competitive with the times.

## Another example

Another example, if you want to use this library http://yohan.giarel.li/Finite/index.html, you have to put a dependency with
Finite\StatefulInterface only because your class has to have the functions `getFiniteState` and `setFiniteState`, this is good and it works
but hard-coding namespaces and dependencies is not the best option.


## What do you think about it?


## EDIT

**That's a RFC for that**

Giorgio told me about this RFC: [php.net/rfc/protocol_type_hinting](https://wiki.php.net/rfc/protocol_type_hinting).

