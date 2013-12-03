---
layout: post
title: "The meaning of Cache for a web developer no more too many connection"
description: "Cache definitions and implementations"
category: post
tags: [HTTP, cache, php, APC, redis, assets]
published: true
edit-link: https://github.com/liuggio/liuggio.github.com/edit/master/_posts/2013-02-22-the-meaning-of-cache-for-a-web-developer-no-more-too-many-connection.md
---
{% include JB/setup %}


In the place of Mr. Obvious I'm going to describe what really cache means, there is a lot of confusion between what the cache **is**, what it **does** and all the possible **uses** and **implementations**.

All the fuss comes from the use**s** of that polysemous word.

## Definition:

We should start from **Wikipedia**, “Cache is a component that transparently stores data so that future requests for that data can be served faster.”

So instead of doing a time consuming calculation if you already 'remember' the result answer it.

The concept seems simple, the problem is it could be used in a myriad of ways.

Important words:

- A **Cache Hit** happens if the Cache contains the answer.

- A **Cache Miss** happens if the Cache doesn’t contain the answer.


## Some 'standard' Code:

**Doctrine** defines the cache interface with  [Doctrine/Common/Cache/Cache.php](https://github.com/doctrine/cache/blob/master/lib/Doctrine/Common/Cache/Cache.php)

    function fetch($id);
    function contains($id);
    function save($id, $data, $lifeTime = 0);
    function delete($id);

**PHP fig-standards** proposed the cache with [proposed/PSR-Cache.md](https://github.com/php-fig/fig-standards/issues?labels=Cache&page=1&state=open)

## Implementations:

As web developers the cache is an essential tool, we have to divide the cache features from its implementations.

### 1. HTTP Cache

#### Description

OK - describing the HTTP Cache is a huge task, and heaps of great developers already did this ([google](https://www.google.com/search?q=http+cache), [fabpot/caching-on-the-edge](http://www.slideshare.net/fabpot/caching-on-the-edge)),
the concept in few lines is that you are a web developer, and the web works with the HTTP Protocol, so you HAVE to study the HTTP Protocol.

I won't go in depth with HTTP Protocol and Cache, but understanding the HTTP protocol turns a developer in a web developer.

The actors in the HTTP Cache are: your web-server, your reverse proxy (as Varnish or Squid), the Shared cache server**s**  (more than one) and the client browser cache.

This is the best Cache and the most powerful, you could create many cache layers on your application, but
if you set up the HTTP headers properly the first time, the next requests will be served for free.
**The least expensive query is the query you never had**

#### Enemies

An enemy of your mental health is the **invalidation**, do not waste time trying to find a solutions for invalidating cache,
choose only the best validation.

The HTTP cache enemy? Cookies and Sessions (see how to solve it thanks to [ESI](https://www.google.it/search?client=ubuntu&channel=fs&q=ESI+cache&ie=utf-8&oe=utf-8&redir_esc=&ei=P6QnUbS5AsbKtAbdQQ))

#### Best Tools

Varnish / Squid

### 2. Opcode

#### Description

“APC is a performance-enhancing extension. It should not be confused with a magic pill, although having it around does provide a positive impact on performance!
 If configured incorrectly, APC can cause unexpected behaviour, however when implemented optimally APC can be a useful weapon in your arsenal.”
[read more on understanding APC](http://techportal.inviqa.com/2010/10/07/understanding-apc/).

This should be a must for you my dear PHP developer.

#### Best Tools

APC o Zend Optimizer+.

### 3. The Cache of your application / framework

#### Description

The framework is a tool that should helps you sort and easily create the logic of your domain,
to do this it provides files and configurators, which are usually elaborated in PHP files,
the cache is used to not process at each request.

For example with Symfony2 in the application cache folders you will find, Url creator files, Proxy Classes, Configuration files (ini, yml, xml) converted to PHP files, Annotations etc...

#### Best Tools

Configuration file to PHP then any Opcode.

### 4. Template and File System Caching

#### Description

Good Template engines, Smarty and Twig provide a caching system using precompiled pieces of code already in a file into the file system.

#### Best Tools

There is no better cache then the Opcode for serving this type of file.

Recently a Pull Request has also been refused, it allows Twig to use any other type of cache [issue on github](https://github.com/fabpot/Twig/issues/728).

### 5. Query Results

#### Description

You are Object Oriented programmers, do not waste your time normalizing the database, if you have too many JOIN delegated the speed to the cache layer.

Queries can be large objects, better to put a centralized server,
The enemy is the period of validity vs. freshness, and the size of the query.

#### Best Tools

Redis/Memcache(d), but [PostgreSQL-cache](http://www.slideshare.net/uptimeforce/postgresql-query-cache-pqc) has a great internal cache layer, also MySQL has it.

### 6. Query Caching

#### Description

Some ORM as Doctrine stores the query creation in order to not process it twice.

#### Best Tools

APC or Redis/Memcache(d)

## Simple Rules

1. If you want to not process content of ini files or xml files or yml files, you could convert to PHP or you could store data see **3.**
2. If you want to cache PHP files use Opcode caching.
3. If you want to store variable contents or Objects use Redis, Memcache(d), or you could also use APC [apc-store](http://php.net/manual/en/function.apc-store.php) (suggested if you have a single web server)
4. If you have provide some content use HTTP Cache always and in order to serve assets (file css, js, static content) use some famous CDN or Nginx or some free/cheap cdn service on the net.

## Other Cache mechanisms

- Pre-Caching/Cacheback, sometimes you need to have some data already available and you don't want that the first request wait some slow task [cacheback-asynchronous-cache-refreshing-for-django](http://codeinthehole.com/writing/cacheback-asynchronous-cache-refreshing-for-django)

- [Microcaching](http://www.howtoforge.com/why-you-should-always-use-nginx-with-microcaching), few seconds of cache, useful for GET or **POST**.
 I said **POST**, do not it seemed strange to you? If you jumped on the chair is ok, POST on cache is insane, unless you do not want to avoid double click.

- Before there were static website in HTML, then dynamic languages ​​and databases, and we finally came to the static pages again, this blog has been developed with [jekyll](http://jekyllbootstrap.com/lessons/jekyll-introduction.html),
