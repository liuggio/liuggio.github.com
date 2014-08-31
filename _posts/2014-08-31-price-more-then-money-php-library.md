---
layout: post
title: "Price more then Money - PHP library"
description: "A Price is the amount in different currencies."
category: post
published: true
tags: [DDD, value objects, money, price]
edit-link: https://github.com/liuggio/liuggio.github.com/blob/master/_posts/2014-08-31-price-more-then-money-php-library.md
---
{% include JB/setup %}

## Abstract 

When we want to put a new open-source library it should be a useful library, it should not exist it the ecosystem, but we should put libraries that explain a good practice, when we satisfy this we are adding a value.

## You need the Money lib.

### Down with Float

In the different versions of our e-commerce we first have used the float to represent the money, yes it was a bad choise, PHP is very bad with all the float operations, shortly we started using BC-math, yet bad choice it was too difficult and too instable.

### Long live Value Object

I already talked about the importance of the [value objects and how to implement the Money](persist-the-money-doctrine-value-object/).

Shortly, how to represent the money information of a product like a `T-Shirt`?

    // bad example
    $amount = '40.0';
    $currency = 'EUR';


As already you know the value object is the answer, read more at [Martin Fowler’s PoEAA](http://martinfowler.com/books.html),
and the [Money lib of Mathias Verraes](https://github.com/mathiasverraes/money) is a good implementation:


    // Using the Verraes Library
    $fiveEur = Money::EUR(500);
    $tenEur = $fiveEur->add($fiveEur);

## You need a Price.

There is always a moment when you need to work on a product that hase different amount on different currencies, especially if you need to implement an e-commerce sooner or later you need different values.

So how to implement the object Price?

### Down with the arrays

If a price calculator must return the price of a T-shirt in both Euro and Dollar, 

surely the array is not the best choice:

    // bad example
    function HowMuchItCosts()
    {
        return array(
	    'EUR' => Money::EUR(500),
        'USD' => Money::USD(550)
        );
    }


### The value object

We have relased the [Price library](https://github.com/leaphly/price):

	composer require leaphly\price 1.0-dev;

#### Simple usage

    The T-Shirt costs 10€ and 8£

In PHP:

	$ticketPrice = new Price(
	  [
	    'EUR' => 1000,
	    'GBP' => 800
	  ]
	);

	echo $tShirtPrice->inEUR();  // return 1000

	var_dump($ticketPrice->availableCurrencies()); // array with EUR, GBP


### A Price is the amount in different currencies.

#### Math operations

Has a different mathematical operators:

    $ticketPrice = new Price(
        [
        'EUR' => 1000,
        'USD' => 1300
        ]
    );

    $discount = new Price(
	    [
	    'EUR' => 10,
	    'USD' => 13
	    ]
    );

    $ticketPrice = $ticketPrice->subtract($discount);


### the API 

the API is in alpha release:

- inXXX magic call eg. `inEUR`() alias of getAmount('EUR')
- `getConversions`
- `isZero`
- `hasAmount`
- `getAmount`
- `availableCurrencies`
- `add`
- `equals`
- `subtract`
- `multiply`
- `divide`

### Working with Conversions

Happens that you have only one fixed amount eg. 10 EUR, 
and you want to provide the converted amount for another currency,
that's a good way to do it:


	$price = new Price(
	    array(
	        'EUR' => 100,
	    ),
	    array('EUR/USD 1.500') // ISO.
	);

	echo $price->inUSD(); // will return 150
	var_dump($price->availableCurrencies()); // will return array('EUR', 'USD')


### Stabilization process

The library is not stable, there are few issues opened, this value object is not immutable, when the issues will be closed, the `Price` library will have a new release.
