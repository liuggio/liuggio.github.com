---
layout: post
title: "Price: more then Money - PHP library"
description: "A Price is the amount in different currencies."
category: post
published: true
tags: [DDD, value objects, money, price]
edit-link: https://github.com/liuggio/liuggio.github.com/blob/master/_posts/2014-08-31-price-more-then-money-php-library.md
---
{% include JB/setup %}

## Abstract 

When we want to put a new open-source library it should be a useful library, it should not exist it the ecosystem, but we should put libraries that explain a good practice, 

when we satisfy this we are adding a value.

## You need the Money lib.

### Down with Float

In the different versions of our e-commerce we first have used the float to represent the money, yes it was a bad choice,

PHP is very bad with all the float operations, shortly we started using BC-math, yet bad choice it was too difficult and too instable.

### Long live Value Object

I already talked about the importance of the [value objects and how to implement the Money](persist-the-money-doctrine-value-object/).

Shortly, how to represent the money information of a product like a `T-Shirt`?

    // bad example
    $amount = '40.0';
    $currency = 'EUR';


As you already know the value object is the answer, read more at [Martin Fowler’s PoEAA](http://martinfowler.com/books.html),
and the [Money lib of Mathias Verraes](https://github.com/mathiasverraes/money) is a good implementation:


    // Using the Verraes Library
    $fiveEur = Money::EUR(500);
    $tenEur = $fiveEur->add($fiveEur);

## You need a Price.

There is always a moment when you need to work on a product that has different amount on different currencies, especially if you need to implement an e-commerce sooner or later you need different values.

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

We have released the [Price library](https://github.com/leaphly/price):

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

##### Constructor with Money

If you already use the Money V.O. in your project the Price constructor also supports Money.

    use Money\Money;

    $price = new Price(
        array(
            Money::EUR(5),
            Money::USD(10),
            'GBP' => 120  // or mixed
        )
    );

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

the APIs are in `-alpha` release:

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
that's a good way to do it, using a conversion ISO:


	$price = new Price(
	    array(
	        'EUR' => 100,
	    ),
	    array('EUR/USD 1.500') // ISO
	);

	echo $price->inUSD(); // will return 150
	var_dump($price->availableCurrencies()); // will return array('EUR', 'USD')


`EUR/USD 1.500` means that if you need the USD amount the Price object will convert starting from the EUR value.

More info at [Currency_pair](http://en.wikipedia.org/wiki/Currency_pair), and [mathiasverraes/CurrencyPair.php](https://github.com/mathiasverraes/money/blob/master/lib/Money/CurrencyPair.php).

### Infrastructure Layer?

Um, it's up to you to understand if you need to persist the Price value object, is also up to you to choose which type of database and how to represent the Price in the database, **but** take a look to `src/Infrastructure/DoctrineORMPriceType.php` is the `Doctrine >2.0` custom type, it uses a string to store the value:

    "EUR 1000;EUR/USD 1.3"

Note: we are not suggesting a relational database as storage or doctrine as ORM, we are only providing a custom type if you decide to use Doctrine 2.0 dbal.

**Enjoy** [github/leaphly/price](https://github.com/leaphly/price).