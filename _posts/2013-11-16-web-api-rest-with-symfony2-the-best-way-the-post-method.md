---
layout: post
title: "part2 - Web API REST with Symfony2"
description: "REST API tutorial on symfony2 second part"
category: tutorial
tags: [rest, api, symfony2]
published: true
date: 2013-11-16 13:00:00
edit-link: https://github.com/liuggio/liuggio.github.com/edit/master/_posts/2013-11-16-web-api-rest-with-symfony2-the-best-way-the-post-method.md
---
{% include JB/setup %}

### Part 2 - the `POST`

In the '[Symfony2 REST part 1](http://welcometothebundle.com/symfony2-rest-api-the-best-2013-way/)' we created the application,
the bundle, we talked about the `GET` method,
we also talked about the importance of the Interfaces, the content negotiation, and we gave an example of dumb controllers and brain services.

Here you could find the [last article Part3 - The rest of REST](/symfony2-rest-api-the-best-way-part-3/)

In this blog post we are going to create a new `Page` via **REST API**: the form is the protagonist of this article.

**Edited**

- 3/12/2013: Added the Request in the actions, thanks to [stloyd](http://twitter.com/stloyd)

## The github repository

There's a repository at [liuggio/symfony2-rest-api-the-best-2013-way](https://github.com/liuggio/symfony2-rest-api-the-best-2013-way/)
you could see the working code using the tag `part2` with

    php composer.phar create-project liuggio/symfony2-rest-api-the-best-2013-way blog-rest-symfony2 -sdev
    cd blog-rest-symfony2
    git checkout -f part2
    bin/phpunit -c app

All the tags for the demo project are at [tags](https://github.com/liuggio/symfony2-rest-api-the-best-2013-way/tags), and also you could compare the first 2 articles with [compare/part1...part2](https://github.com/liuggio/symfony2-rest-api-the-best-2013-way/compare/part1...part2).

## HTTP-bang theory

Just few concepts to know before coding.

<blockquote>
Miyagi: Wax on... wax off. Wax on... wax off.
<small>Karate Kid</small>
</blockquote>

### The HTTP Methods

The HTTP Methods are well described by [github v3 api](http://developer.github.com/v3/) 

Where possible, API strives to use appropriate HTTP verbs for each action.

 - HEAD Can be issued against any resource to get just the HTTP header info.

 - GET  Used for retrieving resources.

 - POST     Used for creating resources, or performing custom actions (such as merging a pull request).

 - DELETE   Used for deleting resources.

 - PATCH    Used for updating resources with partial JSON data. For instance, an Issue resource has title and body attributes. A PATCH request may accept one or more of the attributes to update the resource. PATCH is a relatively new and uncommon HTTP Methods.

 - PUT  Used for replacing resources or collections.

### Safe and idempotent:

**Safe methods**

<blockquote>
    In particular, the convention has been established that the GET and HEAD methods SHOULD NOT have the significance of taking an action other than retrieval. These methods ought to be considered "safe". This allows user agents to represent other methods, such as POST, PUT and DELETE, in a special way, so that the user is made aware of the fact that a possibly unsafe action is being requested. 
    <small> w3c.org <cite title="Source Title">w3c-1 w3.org protocols rfc2616-sec9</cite></small>
</blockquote>

**Idempotent methods**

<blockquote>
    Methods can also have the property of "idempotence" in that (aside from error or expiration issues) the side-effects of N > 0 identical requests is the same as for a single request. The methods GET, HEAD, PUT and DELETE share this property. Also, the methods OPTIONS and TRACE SHOULD NOT have side effects, and so are inherently idempotent.
    <small> w3c.org <cite title="Source Title">w3c-1 w3.org protocols rfc2616-sec9</cite></small>
</blockquote>

So idempotent is about the state of the system, if I create a new resource with POST, the state of the system change every time I call the same Post.

If I delete a resource with $id=10, the system goes always to the same state, so for example multiples and concurrent requests with DELETE pages/10 are accepted without 'side effect'. 

The Safe methods are very important for HTTP-Caching, and 
idempontent is about the Request, the response could be different, the code and the message could change. 

You'd see how the response is about communication and request is about action.

### Verbs and nouns

Since you want to follow the REST methodology, you should create a web interface,
which it uses the HTTP Methods that are available.

The good practices suggests to use **nouns** not verbs (this is not really always true we'll see in the last part of this trilogy).

The convention also imposes to use plurals nouns, `pages` instead `page`, is simpler and coherent.

<table class="table">
<thead>
<tr>
    <th>Resource</th>
    <th>GET</th>
    <th>POST</th>
    <th>PUT</th>
    <th>PATCH</th>
    <th>DELETE</th>
</tr>
</thead>
<tbody>
<tr>
    <td><b>pages</b></td>
    <td>List of Pages</td>
    <td>Create a new Page</td>
    <td>-</td>
    <td>-</td>
    <td>[Delete all the pages]*</td>
</tr>
<tr>
    <td><b>pages/{id}</b></td>
    <td>Show pages/10</td>
    <td>-</td>
    <td>Update a specific page <br>[and create if not exists]*</td>
    <td>Partial update a specific page</td>
    <td>Delete a specific page</td>
</tr>
</tbody>
</table>

The `POST` should create a new resource, the `PUT` should modify an entity.

The `PUT` should also create the resource if it not exists, as the definition of `idempotent`.

But some (quite a lot) web APIs simplify the objective of the PUT (usually they are rubist), giving to it only the update action of a given resource, because is simpler separate the create with `POST` and the update with `PUT`, but the real difference is well explained by the RFC:

<blockquote>
    The fundamental difference between the POST and PUT requests is reflected in the different meaning of the Request-URI. The URI in a POST request identifies the resource that will handle the enclosed entity. That resource might be a data-accepting process, a gateway to some other protocol, or a separate entity that accepts annotations. In contrast, the URI in a PUT request identifies the entity enclosed with the request -- the user agent knows what URI is intended and the server MUST NOT attempt to apply the request to some other resource. If the server desires that the request be applied to a different URI,
</blockquote>

### The actions

The table below, describes the name of the actions, on the head of the table the HTTP Methods.

<table class="table">
<thead>
<tr>
    <th>Resource</th>
    <th>GET</th>
    <th>POST</th>
    <th>PUT</th>
    <th>PATCH</th>
    <th>DELETE</th>
</tr>
</thead>
<tbody>
<tr>
    <td><b>pages</b></td>
    <td>List of Pages
<br>
    getPagesAction()
    </td>
    <td>Create a new Page<br>
    postPagesAction()
    </td>
    <td>-</td>
    <td>-</td>
    <td>-</td>
</tr>
<tr>
    <td><b>pages/{id}</b></td>
    <td>Show
<br>
    getPageAction($id)</td>
    <td>-</td>
    <td>Update
<br>
    putPageAction($id) </td>
    <td>Partial update
<br>
    patchPageAction($id)</td>
    <td>Delete
<br>deletePageAction($id)
    </td>
</tr>
</tbody>
</table>

So just creating the action properly we will have automatically configured the routes, with the proper HTTP Methods.

More info at [rest-action fosRestBundle](https://github.com/FriendsOfSymfony/FOSRestBundle/blob/master/Resources/doc/5-automatic-route-generation_single-restful-controller.md#rest-actions).

## The creation

Back to our Page example, we need a REST API that allows the creation of a Page.

## Step 1 Writing the story

The story: calling the resource `/api/v1/pages.json` with POST method, 
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
and giving a not valid serialized content of the `Page` entity,
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

the **form** is responsible to validate and hydrate the new `Page` object,
then this object is persisted to the Object Manager.

## Step 2 The Entity validation

We have to add the validation layer that will be invisible, the form will perform it automatically:

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

## Step 3 The PageHandler::post

In the first article we created a service called `PageHandler`, its objective is to respect the `PageHandlerInterface`, serving the `Page` entity with `get(id)` and `post()`: it could read and write a `Page` resource.

The first step should be create a test that respects the behavior that we want for the post.

Given parameters containing properties of the `Page` like `array('title'=>'title')`
the function should return an already persisted object respecting the interface `PageInterface`. 

### Step 3.a The post function

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

### Step 3.b The form factory

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

### Step 3.c The processForm

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
     * @throws \Acme\BlogBundle\Exception\InvalidFormException
     */
    private function processForm(PageInterface $page, array $parameters, $method = "PUT")
    {
        $form = $this->formFactory->create(new PageType(), $page, array('method' => $method));
        $form->submit($parameters, 'PATCH' !== $method);
        if ($form->isValid()) {

            $page = $form->getData();
            $this->om->persist($page);
            $this->om->flush($page);

            return $page;
        }

        throw new InvalidFormException('Invalid submitted data', $form);
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

## Step 4 The POST and the Controller

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
     *  statusCode = Codes::HTTP_BAD_REQUEST,
     *  templateVar = "form"
     * )
     *
     * @param Request $request the request object
     *
     * @return FormTypeInterface|View
     */
    public function postPageAction(Request $request)
    {
       try {
           // Hey Page handler create a new Page.
           $newPage = $this->container->get('acme_blog.page.handler')->post(
               $request->request->all()
           );

           $routeOptions = array(
               'id' => $newPage->getId(),
               '_format' => $request->get('_format')
           );

           return $this->routeRedirectView('api_1_get_page', $routeOptions, Codes::HTTP_CREATED);
       } catch (InvalidFormException $exception) {

           return $exception->getForm();
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

Maybe it has gone unnoticed but in the annotation of the documentation, the input of the post is not the `Page` object but a form type:

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

	public function postPageAction(Request $request)
    {
        try {
            $form = new PageType();
            $newPage = $this->container->get('acme_blog.page.handler')->post(
                    $request->request->get($form->getName())
            );
    // ...

## Rest is (also) for human

### People (and REST) got the power

There are many protocols and each has its advantages and its reasons, SOAP, XML-RPC. The added value of REST is that it is not just for client API and web browser is **for people too**.

## Conventional Actions

The title `Rest is also for human` is self explanatory and the guys at Friends Of Symfony, 
wrote how to increase the interaction with the REST process, adding some `conventional actions`:

<blockquote>
HATEOAS, or Hypermedia as the Engine of Application State, is an aspect of REST which allows clients to interact with the REST service with hypertext - most commonly through an HTML page. There are 3 Conventional Action routings that are supported by this bundle: 

<b>new</b>, <b>edit</b>, <b>remove</b> ... from FOSREST-1
</blockquote>


## Step 5 The newPage action

We are going to create the new action:

**new** - A hypermedia representation that acts as the engine to POST. Typically this is a form that allows the client to POST a new resource. Shown as PageController::newPagesAction()

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

We hardcoded the classes inside another class, we create an hard dependency.

### The solution

we could create the PageType as service, and use the name of the service from the PageHandler.
	
    <parameter key="acme_blog.page.type.class">Acme\BlogBundle\Form\PageType</parameter>
    <parameter key="acme_blog.page.type.name"></parameter>
 

    <service id="acme_blog.page.type" class="Leaphly\CartBundle\Form\Type\CartFormType">
        <argument>%acme_blog.page.type.class%</argument>
        <tag name="form.type" alias="leaphly_cart" />
    </service>

and then in the `Page handler` do something like:

	$this->formFactory->createNamed ...

the code given on github **doesn't cover** the form type as service.

### Justifying this extra effort

In few cases this could be considered an [over-engineered](http://en.wikipedia.org/wiki/Overengineering) task,

but when the form type needs extra powers(eg. database access) or when we want to remove all the hard-coding and explicit classes,
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
    public function postPageAction

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

## Next ›› [The last REST API episode](/symfony2-rest-api-the-best-way-part-3/)

In the next articles, we will describe how to delete, edit partially and totally a resource with `DELETE`, `PATCH`, `PUT`,
we will talk how use other important HTTP headers, we will play with `filter` ...

### References

1. [W3C-1  w3.org/Protocols/rfc2616/rfc2616-sec9](http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html)
2. [ZAC-1 - Zach Holman: The Human is a RESTful Client](http://zachholman.com/2010/03/the-human-is-a-restful-client/)
3. [FOSREST-1 conventional-actions](https://github.com/FriendsOfSymfony/FOSRestBundle/blob/master/Resources/doc/5-automatic-route-generation_single-restful-controller.md#conventional-actions)
4. [symfony.com/doc/current/cookbook/form/direct_submit.html](http://symfony.com/doc/current/cookbook/form/direct_submit.html)
5. [symfony.com/doc/current/cookbook/form/create_custom_field_type.html](http://symfony.com/doc/current/cookbook/form/create_custom_field_type.html)
6. [NelmioApiDocBundle](https://github.com/nelmio/NelmioApiDocBundle)
7. [lsmith77/symfony-rest-edition/](https://github.com/lsmith77/symfony-rest-edition/)

