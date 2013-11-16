---
layout: post
title: "Symfony2 REST API: the best way"
description: "REST API tutorial on symfony2"
category: tutorial
tags: [rest, api, symfony2]
date: 2013-11-13 13:00:00
---
{% include JB/setup %}

### Part 1 - the `GET`

Here's another nice guide on how to create an API with Symfony2, this is the **part 1** of a series of articles.

I would like to be short and concise bringing practical examples.

I would not talk about the difference between REST and RESTful [Martin Fowler Maturity Model](http://martinfowler.com/articles/richardsonMaturityModel.html).

The title of this series is just because I've found a lot of great ideas from the [William Durand: rest-apis-with-symfony2-the-right-way](http://williamdurand.fr/2012/08/02/rest-apis-with-symfony2-the-right-way/) blog written in 2012, so this is a revisited version, talking more about form, and services.

## Motivation

Writing Leaphly [symfony cart rest](http://leaphly.org) we had few problems finding a tutorial or a blog post that could show us how to properly use REST and symfony2 with forms and doctrine.

## GOAL

We are going to create an application that serves API for Page content with `get`, `put`, `post` and `patch`, using [Symfony2](http://www.symfony2.com), the [FOSRestBundle](https://github.com/FriendsOfSymfony/FOSRestBundle), the [NelmioApiDocBundle](https://github.com/nelmio/NelmioApiDocBundle), the [JSMSerializerBundle](https://github.com/schmittjoh/JMSSerializerBundle), and [Doctrine](http://www.doctrine-project.org).

The real objective is create an application that shows some best practices and rules with Symfony2 and REST:

### Rules

1. Interface as contracts.
2. Thin Controller, Fat Service.
3. The `Content Negotiation` in the HTTP and REST.
4. Form as API interface.

## The github repository

There's a repository at [liuggio/symfony2-rest-api-the-best-2013-way](https://github.com/liuggio/symfony2-rest-api-the-best-2013-way/)
you could see the working code using the tag `part1` with

    php composer.phar create-project liuggio/symfony2-rest-api-the-best-2013-way blog-rest-symfony2 -sdev
    cd blog-rest-symfony2
    git checkout -f part1
    bin/phpunit -c app

All the tags for the demo project at [tags](https://github.com/liuggio/symfony2-rest-api-the-best-2013-way/tags)

## Step 1.A The application

Create a Symfony application

    php composer.phar create-project symfony/framework-standard-edition BlogRESTAPI/
    cd BlogRESTAPI
    php composer.phar require "friendsofsymfony/rest-bundle" "@dev"
    php composer.phar require "jms/serializer-bundle" "@dev"
    php composer.phar require "nelmio/api-doc-bundle" "@dev"

then we had to configure the bundles properly and add to the appKernel.php

    // app/AppKernel.php
    $bundles = array(
        //..
        new FOS\RestBundle\FOSRestBundle(),
        new JMS\SerializerBundle\JMSSerializerBundle(),
        new Nelmio\ApiDocBundle\NelmioApiDocBundle(),

## Step 1.B The Blog Bundle

We are going to create a REST controller for the Page Entity,
in your symfony2 standard application we need to create the bundle:

    app/console generate:bundle --namespace=Acme/BlogBundle --dir=src --no-interaction

## Step 1.C The Model

We are going to create an Entity called `Page` with `text` and `body`:

    php app/console doctrine:generate:entity --entity=AcmeBlogBundle:Page \
      --format=annotation --fields="title:string(255) body:text" \
      --no-interaction
    php app/console doctrine:database:create
    php app/console doctrine:schema:create

## Step 1.D The Page Form

Now we need the form for that entity, another generator command :)

    php app/console doctrine:generate:form AcmeBlogBundle:Page --no-interaction

## Step 2 - Start with Rest

### Step 2.A - The functional test

We want to create a function that when it's called it returns the resource
with the format requested.

Any good controller should start with a Functional test, but in order to reduce the verbosity I'll talk about functional test later, see the section 3.C below.


    1. /api/v1/pages/{id}.{_format}
    2. /api/v1/pages/{id}.json  # will return a json file
    3. /api/v1/pages/{id}.xml   # will return a xml file
    4. /api/v1/pages/{id} and /api/1/pages/{id}.html  # will return the web page file

We'll see later how to not explicitly specify the format, and how to use and set-up correctly the content-negotiation using HTTP Headers.


### Step 2.B - The controller 

We want to create the controller class

    // /src/Acme/BlogBundle/Controller/PageController.php
    class PageController extends FOSRestController

and then we add a simple and dirty function (we'll refactor soon)

    public function getPageAction($id)
    {
        return $this->container->get('doctrine.entity_manager')->getRepository('Page')->find($id);
    }

#### **[EDIT-14/11/2013]** Samuel Gordalina suggests:

You can use ParamConverter which fetches an entity from database or returns a 404 exception.
For more info see [sample-twitter-api-symfony2:37](https://github.com/gordalina/sample-twitter-api-symfony2/blob/master/src/Twitter/ApiBundle/Controller/TweetController.php#L37)

### Step 2.C - Adding the routes

Add to the route file 
   
    # /app/config/routing.yml
    acme_blog:
        type: rest
        prefix: /api
        resource: "@AcmeBlogBundle/Resources/config/routes.yml"

Create a route file into the bundle:

    # /src/Acme/BlogBundle/Resources/config/routes.yml
    acme_blog_Page:
        type: rest
        prefix: /v1
        resource: "Acme\BlogBundle\Controller\PageController"
        name_prefix:  api_1_ # naming collision

check all the API routes with:

    app/console route:debug | grep api

it should contain the `api_1_get_page`

So we now have a route, and a controller that responses to the get(id) with the Page resource,
is that what we wanted?

Yes but we could do better.

## Step 3 - Refactoring

OMG we have only created a little Controller, why we need to refactor?

As I said we are trying to do things at our best, while this may seem over-engineering, 
in later articles we will see how take advantage of the changes made.

### Step 3.A - Interface as contract


From [symfony.com](http://symfony.com)

<blockquote>
Type hinting the injected object means that you can be sure that a suitable dependency has been injected.
By type-hinting, you'll get a clear error immediately if an unsuitable dependency is injected.
By type hinting using an interface rather than a class you can make the choice of dependency more flexible.
And assuming you only use methods defined in the interface, you can gain that flexibility and still safely use the object.
</blockquote>

Following this as first rule, we need to create an interface in `/src/Acme/BlogBundle/Model/PageInterface.php` and then put `implements PageInterface` in the entity `Page`.

### Step 3.B - The Page Handler

In order to remove all the logic from the `PageController`,
we have to create a service, we call it `PageHandler` in `/src/Acme/BlogBundle/Handler/PageHandler.php`.

The test for the the `PageHandler` looks something like:

    // /src/Acme/BlogBundle/Tests/Handler/PageHandlerTest.php:45
    public function testGet()
    {
        $id = 1;
        $page = $this->getPage(); // create a Page object
        // I expect that the Page repository is called with find(1)
        $this->repository->expects($this->once())
            ->method('find')
            ->with($this->equalTo($id))
            ->will($this->returnValue($page));

        $this->pageHandler->get($id); // call the get.
    }

So it uses `find` to fetch an `id` using the doctrine repository.

We are going to create the effective 'Handler' that will manage all the transactions to the persistence layer:

    // /src/Acme/BlogBundle/Handler/PageHandler.php:16
    class PageHandler implements PageHandlerInterface
    {
        // ..
        public function __construct(ObjectManager $om, $entityClass)
        {
            $this->om = $om;
            $this->entityClass = $entityClass;
            $this->repository = $this->om->getRepository($this->entityClass);
        }

        // ...
        public function get($id)
        {
            return $this->repository->find($id);
        }
    }

We need to make this class available as a service from the dependency injection:

/src/Acme/BlogBundle/Resources/config/services.xml

    <parameters>
        <parameter key="acme_blog.page.handler.class">Acme\BlogBundle\Handler\PageHandler</parameter>
        <parameter key="acme_blog.page.class">Acme\BlogBundle\Entity\Page</parameter>
    </parameters>

    <services>
        <service id="acme_blog.page.handler" class="%acme_blog.page.handler.class%">
            <argument type="service" id="doctrine.orm.entity_manager" />
            <argument>%acme_blog.page.class%</argument>
        </service>
    </services>

### Step 3.C - Thin Controller

Now we have to refactor the controller in order to follow the modification above and use the `PageHandler`.

#### The functional test for the controller:

first add to your `composer.json` the require-dev section:

    "require-dev": {
        "doctrine/doctrine-fixtures-bundle": "dev-master",
        "phpunit/phpunit": "3.7.*",
        "liip/functional-test-bundle":"dev-master"
    },

then we have to update the dependencies running `php composer.phar update`

There's a lot to say about the functional test, we are going to test that when the `api_1_get_page` is called,
it should return a response with `200`, the type of the content should be `json`.

The `liip/functional-test-bundle` helps us
to handle the fixtures data to the persistence layer before each test.

First we configure `fos_rest` in order to handle correct format see: `/app/config/config.yml:69`

We have to create the fixture class see `/src/Acme/BlogBundle/Tests/Fixtures/Entity/LoadPageData.php`

and the test:

    public function testGet()
    {
        $fixtures = array('Acme\BlogBundle\Tests\Fixtures\Entity\LoadPageData');
        $this->customSetUp($fixtures);
        $page = array_pop(LoadPageData::$pages);

        $route =  $this->getUrl('api_1_get_page', array('id' => $page->getId(), '_format' => 'json'));
        $this->client->request('GET', $route);
        $response = $this->client->getResponse();
        $this->assertJsonResponse($response, 200);
        $content = $response->getContent();

        $decoded = json_decode($content, true);
        $this->assertTrue(isset($decoded['id']));
    }

The assertJsonResponse function is well described here: [williamdurand-rest-apis-with-symfony2-the-right-way/#testing](http://williamdurand.fr/2012/08/02/rest-apis-with-symfony2-the-right-way/#testing).

the full test is visible here: /src/Acme/BlogBundle/Tests/Controller/PageControllerTest.php

We have now to modify the function `getPage($id)`:
    
    // /src/Acme/BlogBundle/Controller/PageController.php

    /**
     * Get single Page,
     *
     * @ApiDoc(
     *   resource = true,
     *   description = "Gets a Page for a given id",
     *   output = "Acme\BlogBundle\Entity\Page",
     *   statusCodes = {
     *     200 = "Returned when successful",
     *     404 = "Returned when the page is not found"
     *   }
     * )
     *
     * @Annotations\View(templateVar="page")
     *
     * @param Request $request the request object
     * @param int     $id      the page id
     *
     * @return array
     *
     * @throws NotFoundHttpException when page not exist
     */
    public function getPageAction($id)
    {
        $page = $this->container
            ->get('acme_blog.blog_post.handler')
            ->get($id);

        return $page;
    }

executing the test:

    bin/phpunit -c app

Woow green test!

The bundle looks like:

    src/Acme/BlogBundle/
        ├── AcmeBlogBundle.php
        ├── Controller
        │   └── PageController.php
        ├── DependencyInjection
        ├── Entity
        │   └── Page.php
        ├── Form
        │   └── PageType.php
        ├── Handler
        │   ├── PageHandlerInterface.php
        │   └── PageHandler.php
        ├── Model
        │   └── PageInterface.php
        ├── Resources
        └── Tests
            ├── Controller
            │   └── PageControllerTest.php
            ├── Fixtures
            │   └── Entity
            │       └── LoadPageData.php
            └── Handler
                └── PageHandlerTest.php

## Accessing to the response

Is time to see how the application responses, so executing the php http server

    app/console server:run &

and then accessing to the the resource with `wget`

    wget -S  localhost:8000/api/v1/pages/0.html

We will have a `500` because the database is empty, and that resource doesn't exists:

    --2013-11-09 15:46:37--  http://localhost:8000/api/v1/pages/0.html
    Connecting to localhost (localhost)|127.0.0.1|:8000... connected.
    HTTP request sent, awaiting response... 
      HTTP/1.0 500 Internal Server Error
      Content-type: text/html
    2013-11-09 15:46:37 ERROR 500: Internal Server Error.

The resource '0' doesn't exists,
but we want that the status codes reflects the application behaviour,
so it should return a `404` resource not found.

We are going to create a private function that throws an Exception if the `Page` is not found,
the Exception will modify also automatically the Response Header.

    /**
     * Fetch the Page or throw a 404 exception.
     *
     * @param mixed $id
     *
     * @return PageInterface
     *
     * @throws NotFoundHttpException
     */
    protected function getOr404($id)
    {
        if (!($page = $this->container->get('acme_blog.blog_post.handler')->get($id))) {
            throw new NotFoundHttpException(sprintf('The resource \'%s\' was not found.',$id));
        }

        return $page;
    }

The controller now should use this function and
executing `wget -S  localhost:8000/api/v1/pages/0.html`, we receive a `404` and we are happy :)

## Content Negotiation

An important concept developing the REST API is the [Content Negotiation](http://en.wikipedia.org/wiki/Content_negotiation).

If you think that everything is a resource, maybe you care also about the name of the resource,
if the page `10` is at `/api/v1/pages/10`, you may want to retrieve the same resource with different content type,
not specifying the `format` explicitly in the extension `/api/v1/pages/10.html`, but instead using HTTP `Accept` header.

[EDIT] removed the tags content-negotiation.
If you want to play with the rest application without the extension, set false to `prefer_extension` here:`https://github.com/liuggio/symfony2-rest-api-the-best-2013-way/blob/master/app/config/config.yml#L102`.

Request: `curl -i localhost:8000/api/v1/pages/10`
No Accept header is sent so the fallback is `text/html`

    HTTP/1.1 200 OK
    Host: localhost:8000
    Content-Type: text/html; charset=UTF-8
    Allow: GET

    <html><body><h1>10- title</h2>
    <p>body</p></body></html>

So we retrieve the same resource changing the header:

    curl -i -H "Accept: application/json"  localhost:8000/api/v1/pages/10

Tadaaaam the response is a json file:

    HTTP/1.1 200 OK
    Host: localhost:8000
    Content-Type: application/json
    Allow: GET

    {"id":10,"title":"title","body":"body"}

We could also send different Content type accepted with different preferences eg:

    curl -i -H "Accept: application/json; q=1.0, t/pages/10 q=0.8" localhost:8000/api/v1/

The response will be a json file as well:

    HTTP/1.1 200 OK
    Host: localhost:8000
    Content-Type: application/json
    Allow: GET

    {"id":10,"title":"title","body":"body"}

## Recap

We have created a Doctrine entity called `Page`, we have identified the methods of the interface that will be very useful later on.
We first created a functional test, then the thin controller without logic.
We have created unit test and then the service `PageHandler` which instead of the controller, contains the logic to retrieve the information.
We understood the importance of Content Negotiation.

## Next

In the next articles, we will describe how to use the page form as shared interface, we will create, modify, and delete Pages, with `PUT`, `PATCH`, `POST`, `DELETE`, and we will detail how use other important HTTP headers.


### References:

[Symfony.com](http://www.symfony2.com)

[Lukas Kahwe Smith: resting with Sf2 - video](http://www.youtube.com/watch?v=Kkby5fG89K0&feature=youtu.be&from=www.welcometothebundle.com)

[William Durand: rest-apis-with-symfony2-the-right-way - blog](http://williamdurand.fr/2012/08/02/rest-apis-with-symfony2-the-right-way/)

[Samuel Gordalina: REST APIs made easy with Symfony2 - slide](https://speakerdeck.com/gordalina/rest-apis-made-easy-with-symfony2)

[Daniel Londero: Rest in practice - slide](http://www.slideshare.net/dlondero/rest-in-practice-27335543)
