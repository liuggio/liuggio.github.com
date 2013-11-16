---
layout: post
title: "Web API REST with Symfony2: the best way - part 2 - The POST method"
description: "REST API tutorial on symfony2 second part"
category: tutorial
tags: [rest, api, symfony2]
published: false
date: 2013-11-16 13:00:00
---
{% include JB/setup %}

### Part 2 - the `POST`

In the '[Symfony2 REST part 1](http://welcometothebundle.com/symfony2-rest-api-the-best-2013-way/)' we created the application, the bundle, we talked about the `GET` method, we also talked about the importance of the Interfaces, the content negotiation, and we gave an example of dumb controllers and brain services.

In this blog post we are going to create a new `Page` via REST API: the form is the protagonist of this article.

## The github repository

There's a repository at [liuggio/symfony2-rest-api-the-best-2013-way](https://github.com/liuggio/symfony2-rest-api-the-best-2013-way/)
you could see the working code using the tag `part2` with

    php composer.phar create-project liuggio/symfony2-rest-api-the-best-2013-way blog-rest-symfony2 -sdev
    cd blog-rest-symfony2
    git checkout -f part2
    bin/phpunit -c app

All the tags for the demo project are at [tags](https://github.com/liuggio/symfony2-rest-api-the-best-2013-way/tags), and also you could compare the first 2 articles with [compare/part1...part2](https://github.com/liuggio/symfony2-rest-api-the-best-2013-way/compare/part1...part2).

## The creation

We need a REST API that allows the creation of a Page.

The story:  calling the resource `/api/v1/pages.json` with POST method, 
and giving the whole serialized content of a `Page` entity,
the response should have the status code `201`, and should be a json response.

We could easily write this story into a functional test:
	
	// src\Acme\BlogBundle\Tests\Controller\PageControllerTest
    public function testJsonPostPageAction()
    {
        $this->client = static::createClient();
        $this->client->request(
            'POST', 
            '/api/v1/pages.json',  
            array(),
            array(),
            array('CONTENT_TYPE' => 'application/json'),
            '{"title":"title1","body":"body1"}'
        );

        $this->assertJsonResponse($this->client->getResponse(), 201, false);
    }

and there is another story:

calling the resource `/api/v1/pages.json` with the POST method, 
and giving a not correct serialized content of the `Page` entity,
the response should have the status code `400`.

    public function testJsonPostPageActionShouldReturn400WithBadParameters()
    {
        $this->client = static::createClient();
        $this->client->request(
            'POST',
            '/api/v1/pages.json',
            array(),
            array(),
            array('CONTENT_TYPE' => 'application/json'),
            '{"ninja":"turtles"}'
        );

        $this->assertJsonResponse($this->client->getResponse(), 400, false);
    }

Of course the tests are red, we had to add the `post` function in `PageHandler` then the `postPageAction` in the Controller.

We are going to create a `post` function that takes the parameters (as an array) containing all the fields of the entity `Page`,
the form is responsible to validate and hydrate the new `Page` object,
then this object is persisted to the Object Manager.

## The validation

We have to add a validation layer that will be invisible because the form will perform it automatically:

	# src/Acme/BlogBundle/Resources/config/validation.yml
	Acme\BlogBundle\Entity\Page:
	    properties:
	        title:
	            - NotBlank: ~
	            - NotNull: ~
	            - Length:
	                min: 2
	                max: 50
	                minMessage: "Your title must be at least {{ limit }} characters length"
	                maxMessage: "Your title name cannot be longer than {{ limit }} characters length"

## The PageHandler::post

The first step should be create a test that respects the behavior.

Given parameters containing properties of the `Page` like `array('title'=>'title')`
the function should return an already persisted object respecting the interface `PageInterface`. 

We are now going to create a function that look like even in the name to the controller's function.

	// src/Acme/BlogBundle/Handler/PageHandler.php
	/**
	 * Create a new Page.
	 *
	 * @param array $parameters
	 *
	 * @return PageInterface
	 */
	public function post(array $parameters)
	{
	    $page = $this->createPage(); // factory method create an empty Page

	    // Process form does all the magic, validate and hydrate the Page Object.
	    return $this->processForm($page, $parameters, 'POST');
	}

In order to use the form we need the `form.factory` injected into the service:
	
	// src\Acme\BlogBundle\Handler\PageHandler.php
	use Symfony\Component\Form\FormFactoryInterface;
	// ...
	class PageHandler implements PageHandlerInterface
	// ...
	private $formFactory;
	// ...
	public function __construct(ObjectManager $om, $entityClass, FormFactoryInterface $formFactory)
	{
	    $this->om = $om;
	    $this->entityClass = $entityClass;
	    $this->repository = $this->om->getRepository($this->entityClass);
	    $this->formFactory = $formFactory;
	}

and then we have to add a new argument into the page handler service at `/src/Acme/BlogBundle/Resources/config/services.xml`
	
	<?xml version="1.0" ?>
    <services>
        <service id="acme_blog.page.handler" class="%acme_blog.page.handler.class%">
            <argument type="service" id="doctrine.orm.entity_manager" />
            <argument>%acme_blog.page.class%</argument>
            <argument type="service" id="form.factory"></argument>
        </service>
    </services>

It's time to back to `\Acme\BlogBundle\Handler\PageHandler` and create the `processForm` function, the hearth of the write functions.

You have surely already used [form->handleRequest($request)](http://api.symfony.com/2.3/Symfony/Component/Form/FormInterface.html),
but in this case we don't have the Request but only the values ​​
in an array, so instead of `handleRequest`, we are going to `submit` the form.
    
    /**
     * Processes the form.
     *
     * @param PageInterface $page
     * @param array         $parameters
     * @param String        $method
     *
     * @return PageInterface
     *
     * @throws Acme\BlogBundle\Exception\InvalidFormException
     */
    private function processForm(PageInterface $page, array $parameters, $method = "PUT")
    {
        $form = $this->formFactory->create(new PageType(), $page, array('method' => $method));
        $form->submit($parameters, 'PATCH' !== $method);
        if (!$form->isValid()) {
            throw new InvalidFormException('Invalid submitted data', $form);
        }
        $page = $form->getData(); // get the hydrated Page object
        $this->om->persist($page);// persistence layer
        $this->om->flush($page);

        return $page;
    }

This function returns an object of `PageInterface` type, but if the parameters are not valid it
throws an `InvalidFormException` that will be handled and caught by the controller.

The test class is at [part2-src/Acme/BlogBundle/Tests/Handler/PageHandlerTest.php](https://github.com/liuggio/symfony2-rest-api-the-best-2013-way/blob/part2/src/Acme/BlogBundle/Tests/Handler/PageHandlerTest.php/)

### Why don't use the request?

The real question is: do we really need the request here?

The answer is **no**, the main responsibility of `PageHandler` class is to `handle` (creating, editing, showing)
the `Page` entity, given some parameters, it doesn't care about Request.

In this way you could use `PageHandler` from container services, without injecting or faking the Request.

In the end of this series of articles you will have an application that serves REST API,

but the PageHandler is itself an API, usable by services of your application via service container.

## The POST and the Controller

### The function

The postPageAction is simple, all the domain logic is demanded to the `PageHandler`.

    /**
     * Create a Page from the submitted data.
     *
     * @ApiDoc(
     *   resource = true,
     *   description = "Creates a new page from the submitted data.",
     *   input = "Acme\BlogBundle\Form\PageType",
     *   statusCodes = {
     *     200 = "Returned when successful",
     *     400 = "Returned when the form has errors"
     *   }
     * )
     *
     * @Annotations\View(
     *  template = "AcmeBlogBundle:Page:newPage.html.twig",
     *  statusCode = Codes::HTTP_BAD_REQUEST
     * )
     *
     * @return FormTypeInterface|RouteRedirectView
     */
    public function postPageAction()
    {
        try {
        	// Hey handler create a page for me.
            $newPage = $this->container->get('acme_blog.page.handler')->post(
                    $this->container->get('request')->request->all()
            );

            $routeOptions = array(
                'id' => $newPage->getId(),
                '_format' => $this->container->get('request')->get('_format')
            );

            // return HTTP_CREATED, and add location header
            return $this->routeRedirectView('api_1_get_page', $routeOptions, Codes::HTTP_CREATED);

        } catch (InvalidFormException $exception) {

            return $this->view(array('errors' => $exception->getForm()), Codes::HTTP_BAD_REQUEST);
        }
    }

### The location

A new page is created by the `PageHandler`, then the helper `routeRedirectView` adds to the http header
 the location: `http://localhost:8000/api/v1/pages/47.json`.

### The template

Remember to create the template newPage.html.twig something like

    /src/Acme/BlogBundle/Resources/views/Page/newPage.html.twig
    <h1>Page Form</h1>
    <form action="{ { url('api_1_post_page') } }" method="POST" { { form_enctype(form) } }>
        { { form_widget(form) } }
        <input type="submit" value="submit">
    </form>

### The API

Maybe it has gone unnoticed but the inputs to the post is not the `Page` object but a form type:

    * @ApiDoc(
    *   ...
    *   input = "Acme\BlogBundle\Form\PageType",
    *   ...

You'd see how much the form is important in our API application.

### The name of the Form type

One thing that seems very obscure in the eyes of those doing the rest with the form is the `getName`,

in all the examples above we sent all the data with an empty `PageType::name` but if we'd change to:

	// src/Acme/BlogBundle/Form/PageType.php
    /**
     * @return string
     */
    public function getName()
    {
        return 'page';
    }

The data should be sent with a different syntax:

	curl -X POST -d '{"page":{"title":"title1","body":"body1"}}' http://localhost:8000/api/v1/pages.json --header "Content-Type:application/json" -v

and the controller should look something like

	public function postPageAction()
    {
        try {
            $form = new PageType();
            $newPage = $this->container->get('acme_blog.page.handler')->post(
                    $this->container->get('request')->request->get($form->getName())
            );
    // ...

## Rest is for human

<blockquote>... There's a number of great technical reasons to move to an architecture like REST.
In fact, there's a number of great reasons to implement SOAP, XML-RPC, or similar architecture on your app, the distinctions being less relevant for this post.
But REST isn't just for API clients or web browsers: REST is for people, too.
... from ZAC-1 </blockquote>

### The newPage action

We are providing a REST API for `json`, `xml`, and `html` formats,
so a user could be able to create a correct request, we should provide also a web page to create the form:

    /**
     * Presents the form to use to create a new page.
     *
     * @ApiDoc(
     *   resource = true,
     *   statusCodes = {
     *     200 = "Returned when successful"
     *   }
     * )
     *
     * @Annotations\View()
     *
     * @return FormTypeInterface
     */
    public function newPageAction()
    {
        return $this->createForm(new PageType());
    }

Automatically a new route is added at `/api/v1/pages/new.{_format}`
you could check it with

 `app/console router:debug | grep api`

Accessing to the the page with `curl -S localhost:8000/api/v1/pages/new`

you will obtain a working html form.

## Manually test the application

### The happy path 201

	curl -X POST -d '{"title":"title","body":"body"}' http://localhost:8000/api/v1/pages.json --header "Content-Type:application/json" 

It will return 

	< HTTP/1.1 201 Created
	< Host: localhost:8000
	< Location: http://localhost:8000/api/v1/pages/50.json
	< Allow: POST
	< Content-Type: application/json

This is great! HTTP Status code `201` resource created, and location tells to the consumers where to get this resource.

### Bad parameters 400

Let's try to send a bad content:

	curl -X POST -d '{"ninja":"title","turtles":"body"}' http://localhost:8000/api/v1/pages.json --header "Content-Type:application/json" -v

The Response will be `400`

	< HTTP/1.1 400 Bad Request
	< Host: localhost:8000
	< Content-Type: application/json
	< 
	< {"form":{"errors":["This form should not contain extra fields."],"children":{"title":[],"body":[]}}}

## Things to know (and not always to do)

There are simple steps that could improve a little bit the project:

### The non-problem

Do you remember how we used the formFactory in the PageHandler?

	// src\Acme\BlogBundle\Handler\PageHandler::$formFactory
	$form = $this->formFactory->create(new PageType(), $page, array('method' => $method));

and do you remember how we created the PageType?

	// src/Acme/BlogBundle/Form/PageType.php
	'data_class' => 'Acme\BlogBundle\Entity\Page',

We hardcoded the classes inside another class, we create a dependency.

### The solution

we could create the PageType as service, and use the name of the service from the PageHandler.
	
	<parameter key="acme_blog.page.type.class">Acme\BlogBundle\Form\PageType</parameter>
    <parameter key="acme_blog.page.type.name"></parameter>
    <parameter key="acme_blog.page.type.alias">leaphly_cart</parameter>

    <service id="acme_blog.page.type" class="Leaphly\CartBundle\Form\Type\CartFormType">
        <argument>%acme_blog.page.type.class%</argument>
        <tag name="form.type" alias="%acme_blog.page.type.alias%" />
    </service>

and then in the `Page handler` do something like:

	$this->formFactory->createNamed ...

the code given on github **doesn't cover** the form type as service.

### Justifying this extra effort

In few cases this could be considered an [over-engineered](http://en.wikipedia.org/wiki/Overengineering) task,

but when the form type needs extra powers or when we want to remove all the hard-coding and explicit classes,
or if we want to centralize the parameters and the classes in order to be easily changed or renamed,
the form type as service could be useful.

## The documentation as bonus

With [NelmioApiDocBundle](https://github.com/nelmio/NelmioApiDocBundle) we will have a documentation **for free**

adding a route:
	
	# app/config/routing.yml
	NelmioApiDocBundle:
	    resource: "@NelmioApiDocBundle/Resources/config/routing.yml"
	    prefix:   /api/doc

then modifying the PostAction annotations

	/**
     * @ApiDoc(
     *   resource = true,
     *   description = "Creates a new page from the submitted data.",
     *   input = "Acme\BlogBundle\Form\PageType",
     *   statusCodes = {
     *     200 = "Returned when successful",
     *     400 = "Returned when the form has errors"
     *   }
     * )
     */
    public function postPageAction()

And thanks to [this guys](https://github.com/nelmio/NelmioApiDocBundle/graphs/contributors) you will have something like:

![NelmioApiDocBundle screen-shot]({{ ASSET_PATH }}/readable-liuggio/img/screenshot-nelmio.png)

## Recap

Now we could create and read a resource via HTTP REST,
using those routes:

    api_1_get_page           GET    ANY    ANY  /api/v1/pages/{id}.{_format}
    api_1_new_page           GET    ANY    ANY  /api/v1/pages/new.{_format}
    api_1_post_page          POST   ANY    ANY  /api/v1/pages.{_format}
    nelmio_api_doc_index     GET    ANY    ANY  /api/doc/

We also have created another API usable via service container and the API are defined into the `PageHandlerInterface`:

    // src/Acme/BlogBundle/Handler/PageHandlerInterface.php
    interface PageHandlerInterface
    {
        /**
         * Get a Page given the identifier
         *
         * @api
         *
         * @param int $id
         *
         * @return PageInterface
         */
        public function get($id);

        /**
         * Post Page, creates a new Page.
         *
         * @api
         *
         * @param array $parameters
         *
         * @return PageInterface
         */
        public function post(array $parameters);
    }

## Next

In the next articles, we will describe how to delete, edit partially and totally a resource with `DELETE`, `PATCH`, `PUT`,
we will detail how use other important HTTP headers, we will play with `filter` and a simple js integration.



### References

[ZAC-1 - Zach Holman: The Human is a RESTful Client](http://zachholman.com/2010/03/the-human-is-a-restful-client/)

[symfony.com/doc/current/cookbook/form/direct_submit.html](http://symfony.com/doc/current/cookbook/form/direct_submit.html)

[symfony.com/doc/current/cookbook/form/create_custom_field_type.html](http://symfony.com/doc/current/cookbook/form/create_custom_field_type.html)

[NelmioApiDocBundle](https://github.com/nelmio/NelmioApiDocBundle)

[lsmith77/symfony-rest-edition/](https://github.com/lsmith77/symfony-rest-edition/)
