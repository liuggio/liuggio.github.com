---
layout: post
title: "Persist the Money, Doctrine Value Object"
description: "Finally is possible to use and persist value object"
category: tutorial
tags: [doctrine, DDD, value objects, money]
edit-link: https://github.com/liuggio/liuggio.github.com/edit/master/_posts/2014-03-21-persist-the-money-doctrine-value-object.md
---
{% include JB/setup %}

Working on large projects, I noticed that we lose too much time thinking about the famous 'M' of the *MVC* pattern 
instead of thinking about the communication of the various modules.

## The magical world of the Value Object

Surely the Value Objects has always been overshadowed by the entity as V.Vernon also says in [his red book](http://www.amazon.it/Implementing-Domain-Driven-Design-Vaughn-Vernon/dp/0321834577).

We are accustomed to think about Entity, and also on how to store those entities in the db, but often what we need is to focus on the Value Objects.

If you want to enter into the magical world of the value object, you could find more detail on internet, the value object is an important concept for the DDD community.

## Identify a Value Object

A simplified recipe to identify an entity from a value object:

if an object needs a life cycle, if an object needs to be identified is an Entity.

If you need an immutable object, that its meaning lies in the value, is a Value Object.

Very Often you could find a Value Object inside an Entity.

## Implementing Money

The most famous example of Value Object is the Money scenario.

	Class Order {
	  /** @var int */
	  protected $id;
	  /** @var int */
	  protected $price;
	  /** @var string */
	  protected $currency;
	  
	  … more proprerties
	}

Does this sound familiar?

Well surely there is a more elegant way to do it, we are objects oriented programmers.

	Class Order {
	  /** @var int */
	  protected $id;
	  /** @var Money */
	  protected $price;
	  
	  // … more proprerties
	}  

   $order->setPrice(new Money(10, 'EUR'));

*Price* is a Value Object!

### Persisting Value Objects in PHP

With Doctrine minor2.5 a custom type is allowed only for one-to-one mapping between field and column of the relational database,
and representing an object with more proprerties is difficult.

A very common approach was to create a custom `MoneyType` see [Doctrine2/MoneyType](https://github.com/mathiasverraes/money/blob/708d8d53b2374e1f9686dceee4f9636df32f6d43/lib/Money/Doctrine2/MoneyType.php), with this custom type Money is converted to a string eg. `Money(100, 'EUR')` would become a `string` of `"1000 EUR"`.

A good approach to represent a Value Object leveraging on its immutability, was to record the value object directly as a serialized [Object type](http://doctrine-orm.readthedocs.org/en/latest/reference/basic-mapping.html#doctrine-mapping-types) in Doctrine.

All the above approaches led to drawbacks in terms of usability especially when you need to queries.

### Embeddables

Finally, we can work the value object with the beautiful feature of [Embeddables](http://doctrine-orm.readthedocs.org/en/latest/tutorials/embeddables.html) available with **Doctrine 2.5**

	/** @Entity */
	class Order
	{
	    /** @Id */
	    private $id;

	    /** @Embedded(class = "Money") */
	    private $money;
	}

	/** @Embeddable */
	class Money
	{
	    /** @Column(type = "int") */ // better then decimal see the mathiasverraes/money documentation
	    private $amount;

	    /** @Column(type = "string") */
	    private $currency;
	}

Is also possible to use DQL to order by the amount or to filter using the fields of the Value Object.

	SELECT o FROM Order o WHERE o.money.currency = :currency


A very used library for handle [Money is here](https://packagist.org/packages/mathiasverraes/money) thanks to [@mathiasverraes](https://twitter.com/mathiasverraes)

That's all, thanks Domain Driven Design and Doctrine community.
