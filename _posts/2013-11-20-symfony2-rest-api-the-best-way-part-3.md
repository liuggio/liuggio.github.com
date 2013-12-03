---
layout: post
title: "part 3 - Symfony2 API: the rest of REST"
description: "REST API tutorial on symfony2 with best practices"
category: tutorial
tags: [rest, api, symfony2]
published: true
date: 2013-12-03 13:00:00
edit-link: https://github.com/liuggio/liuggio.github.com/edit/master/_posts/2013-11-20-symfony2-rest-api-the-best-way-part-3.md
---
{% include JB/setup %}

### Part 3 - the rest of rest.

This is trilogy's final, in the part1 '[best web API with Symfony2](http://welcometothebundle.com/symfony2-rest-api-the-best-2013-way/)' we created the application and the bundle, we wrote the `GET` method, we also talked about the importance of the Interfaces, the content negotiation, and we gave an example of dumb controllers and smart services...

In the part2 '[REST and Symfony the best way](http://welcometothebundle.com/symfony2-rest-api-the-best-2013-way/)' we created a new `Page` via **REST API**, we talked about `idempotent` and `safe` methods, and we understood how much the form is important...

In this article we are going to show how to properly modify a given resource using `PUT`,`PATCH`, how to handle different role for the Page resource, and how to disable the CRSF protection only when using API.

## The github repository

There's a repository at [liuggio/symfony2-rest-api-the-best-2013-way](https://github.com/liuggio/symfony2-rest-api-the-best-2013-way/)
you could see the working code using the tag `part3` with

    php composer.phar create-project liuggio/symfony2-rest-api-the-best-2013-way blog-rest-symfony2 -sdev
    cd blog-rest-symfony2
    git checkout -f part3
    bin/phpunit -c app

All the tags for the demo project are at [tags](https://github.com/liuggio/symfony2-rest-api-the-best-2013-way/tags), and also you could compare the 2 articles with [compare/part2...part3](https://github.com/liuggio/symfony2-rest-api-the-best-2013-way/compare/part2...part3).

## This is a CRUD world

In the second article, we also saw how to properly use the HTTP-verbs, if you have to create a resource you should use the `POST` method,
 if you want to fetch a resource you should use the `GET` method,
 you have all the tools in order to create a CRUD (Create, Read, Update and Delete) with REST.

<blockquote>
The key principles of REST involve separating your API into logical resources. These resources are manipulated using HTTP requests where the method (GET, POST, PUT, PATCH, DELETE) has specific meaning.<small>Best Practices for Designing a Pragmatic RESTful API</small>
</blockquote>

<blockquote>
The great thing about REST is that you're leveraging existing HTTP methods to implement significant functionality on just a single /tickets endpoint. There are no method naming conventions to follow and the URL structure is clean and clear.<small>Best Practices for Designing a Pragmatic RESTful API</small>
</blockquote>

## The update implementation

`PUT` method is used to replace all the content of a resource,
the `PATCH` method is utilized instead to partially update a resource.
 
### PUT that PATCH

As always we need the stories and the tests before coding:

- **PUT** - given a Page, is possible to replace all its proprieties, and if that page doesn't exist should be created.

- **PATCH** - given a Page, is possible to replace some proprieties.

### github's star

If you where github, how did you develop the API in order to add a star to a Gist?

Story: A user should be able to 'star' a gist.

Github api/v3 uses:

 - PUT    /gists/:id/star - to 'star' a gist and it will response a `204 No Content`
 - DELETE /gists/:id/star - to 'unstar' a gist and it will response a `204 No Content`
 - GET    /gists/:id/star - to check if a gist is starred was starred will response a `204 No Content` and return a `404` if it wasn't.

### PUT the implementation

The implementation of the PUT and PATCH methods should be in our service called `PageHandler`, all the job is already done by the `processForm` function that we have developed in the last article.

Adding to `PageHandlerInterface` the new methods, the `PageHandler` will be:

    /**
     * Edit a Page, or create if not exist.
     *
     * @param PageInterface $page
     * @param array         $parameters
     *
     * @return PageInterface
     */
    public function put(PageInterface $page, array $parameters)
    {
        return $this->processForm($page, $parameters, 'PUT');
    }

    /**
     * Partially update a Page.
     *
     * @param PageInterface $page
     * @param array         $parameters
     *
     * @return PageInterface
     */
    public function patch(PageInterface $page, array $parameters)
    {
        return $this->processForm($page, $parameters, 'PATCH');
    }

The function Put doesn't create the resource.

#### So #@* easy?

**yes!**

The functions need a Page object and its parameters, the form will do the magic.

### The Controller

In the controller we have to handle the HTTP status code, the headers, and the page object.

The tests for putAction are two, one for the creation, and one for the modification:

	public function testJsonPutPageActionShouldModify()
    {
        $fixtures = array('Acme\BlogBundle\Tests\Fixtures\Entity\LoadPageData');
        $this->customSetUp($fixtures);
        $pages = LoadPageData::$pages;
        $page = array_pop($pages);

        $this->client->request('GET', sprintf('/api/v1/pages/%d.json', $page->getId()), array('ACCEPT' => 'application/json'));
        $this->assertEquals(200, $this->client->getResponse()->getStatusCode(), $this->client->getResponse()->getContent());

        $this->client->request(
            'PUT',
            sprintf('/api/v1/pages/%d.json', $page->getId()),
            array(),
            array(),
            array('CONTENT_TYPE' => 'application/json'),
            '{"title":"abc","body":"def"}'
        );

        $this->assertJsonResponse($this->client->getResponse(), 204, false);
        $this->assertTrue(
            $this->client->getResponse()->headers->contains(
                'Location',
                sprintf('http://localhost/api/v1/pages/%d.json', $page->getId())
            ),
            $this->client->getResponse()->headers
        );
    }

    public function testJsonPutPageActionShouldCreate()
    {
        $id = 0;
        $this->client->request('GET', sprintf('/api/v1/pages/%d.json', $id), array('ACCEPT' => 'application/json'));
        $this->assertEquals(404, $this->client->getResponse()->getStatusCode(), $this->client->getResponse()->getContent());

        $this->client->request(
            'PUT',
            sprintf('/api/v1/pages/%d.json', $id),
            array(),
            array(),
            array('CONTENT_TYPE' => 'application/json'),
            '{"title":"abc","body":"def"}'
        );

        $this->assertJsonResponse($this->client->getResponse(), 201, false);
    }

And the `PUT` controller will look like:

    /**
     * Update existing page from the submitted data or create a new page at a specific location.
     *
     * @ApiDoc(
     *   resource = true,
     *   input = "Acme\DemoBundle\Form\PageType",
     *   statusCodes = {
     *     201 = "Returned when the Page is created",
     *     204 = "Returned when successful",
     *     400 = "Returned when the form has errors"
     *   }
     * )
     *
     * @Annotations\View(
     *  template = "AcmeBlogBundle:Page:editPage.html.twig",
     *  templateVar = "form"
     * )
     *
     * @param Request $request the request object
     * @param int     $id      the page id
     *
     * @return FormTypeInterface|View
     *
     * @throws NotFoundHttpException when page not exist
     */
    public function putPageAction(Request $request, $id)
    {
        try {
            if (!($page = $this->container->get('acme_blog.page.handler')->get($id))) {
                $statusCode = Codes::HTTP_CREATED;
                $page = $this->container->get('acme_blog.page.handler')->post(
                    $request->request->all()
                );
            } else {
                $statusCode = Codes::HTTP_NO_CONTENT;
                $page = $this->container->get('acme_blog.page.handler')->put(
                    $page,
                    $request->request->all()
                );
            }

            $routeOptions = array(
                'id' => $page->getId(),
                '_format' => $request->get('_format')
            );

            return $this->routeRedirectView('api_1_get_page', $routeOptions, $statusCode);

        } catch (InvalidFormException $exception) {

            return $exception->getForm();
        }
    }

The PatchPageAction and the DeletePageAction will be very similar.

### Manually testing the lifecycle

We are going to reproduce all the page lifecycle putting the value of the result of the post
into a bash variable, so we could reuse. 

	location=`curl -X POST -d '{"title":"liuggio","body":"homepage"}' \
	   http://localhost:8000/api/v1/pages.json \
	   --header "Content-Type:application/json" -v 2>&1 | grep Location | cut -d \  -f 3`;

	echo "created result at: "$location

the result will look something like:

	created result at http://localhost:8000/api/v1/pages/143.json

and we are going to use this $location variable and we will modify the resource:

	curl -X PUT -d '{"title":"my","body":"homepage"}' $location --header "Content-Type:application/json" -v

	curl -X PATCH -d '{"body":"life"}' $location --header "Content-Type:application/json" -v

and then the get:

	curl $location --header "Content-Type:application/json" 

the result will be:

	{"id":140,"title":"my","body":"life"}

## GET all the Pages

We want to be able to get a list of all the pages, we want to limit the result with pagination.

    /**
     * List all pages.
     *
     * @ApiDoc(
     *   resource = true,
     *   statusCodes = {
     *     200 = "Returned when successful"
     *   }
     * )
     *
     * @Annotations\QueryParam(name="offset", requirements="\d+", nullable=true, description="Offset from which to start listing pages.")
     * @Annotations\QueryParam(name="limit", requirements="\d+", default="5", description="How many pages to return.")
     *
     * @Annotations\View(
     *  templateVar="pages"
     * )
     *
     * @param Request               $request      the request object
     * @param ParamFetcherInterface $paramFetcher param fetcher service
     *
     * @return array
     */
    public function getPagesAction(Request $request, ParamFetcherInterface $paramFetcher)
    {
        $offset = $paramFetcher->get('offset');
        $offset = null == $offset ? 0 : $offset;
        $limit = $paramFetcher->get('limit');

        return $this->container->get('acme_blog.page.handler')->all($limit, $offset);
    }

Using the annotation `@Annotations\QueryParam`, is very easy to get the `offset`, and `limit` that is ORM-ready :). 

This is the `PageHandler::all`:

    /**
     * Get a list of Pages.
     *
     * @param int $limit  the limit of the result
     * @param int $offset starting from the offset
     *
     * @return array
     */
    public function all($limit = 5, $offset = 0, $orderby = null)
    {
        return $this->repository->findBy(array(), $orderby, $limit, $offset);
    }

### Pagination with headers

Is possible to include pagination into the headers, there is a RFC: [Link header introduced by RFC 5988](http://tools.ietf.org/html/rfc5988#page-6),
you could find a good implementation on the [github pagination documentation](http://developer.github.com/v3/#pagination).

<blockquote>
<b>Link Header</b><br>
The pagination info is included in the Link header. It is important to follow these Link header values instead of constructing your own URLs. In some instances, such as in the Commits API, pagination is based on SHA1 and not on page number.<small>github link header</small>
</blockquote>

github example: 

    Link: <https://api.github.com/user/repos?page=3&per_page=100>; rel="next",<https://api.github.com/user/repos?page=50&per_page=100>; rel="last"

## Symfony2 goodies 

### HEAD method

In Symfony2 the HEAD method follows the same routes and settings of the GET method,
if you have the GET method available you will also have the `HEAD` that links to the same actions,
the response will be empty.

### Patching the PATCH problem

Some browser doesn't support some methods as `PUT`, `PATCH` and `DELETE` HTTP methods:

<blockquote>
    Fortunately Symfony2 provides you with a simple way of working around this limitation.<br>
     By including a _method parameter in the query string or parameters of an HTTP request,
     Symfony2 will use this as the method when matching routes. <br>
     Forms automatically include a hidden field for this parameter if their submission method is not GET or POST.<br>
      See the related chapter in the forms documentation for more information.
    <small>http://symfony.com/doc/current/cookbook/routing/method_parameters.html#faking-the-method-with-method</small>
</blockquote>

## Disable CSRF with REST

### Theory on Cross-site request forgery 

If you don't know what CSFR is there is lot of documentation on internet, please have a look to the [csrf-protection](http://symfony.com/doc/current/book/forms.html#csrf-protection)

<blockquote>
CSRF protection works by adding a hidden field to your form - called _token by default - that contains a value that only you and your user knows. This ensures that the user - not some other entity - is submitting the given data. Symfony automatically validates the presence and accuracy of this token.
<small>http://symfony.com/doc/current/book/forms.html#csrf-protection</small>
</blockquote>

### CSRF and REST

Is possible to disable the CSRF based on the user's role, the documentation on Friend-Of-Symfony repository is self-explanatory:
[FOSRestBundle doc CSRF validation](https://github.com/FriendsOfSymfony/FOSRestBundle/blob/master/Resources/doc/2-the-view-layer.md#CSRF-validation).

We need to associate the `ROLE_API` to the REST users:

    # app/config/security.yml
    providers:
        in_memory:
            memory:
                users:
                    user:  { password: userpass, roles: [ 'ROLE_USER', 'ROLE_API' ] }
                    admin: { password: adminpass, roles: [ 'ROLE_ADMIN', 'ROLE_API' ] }

then in the config.yml we need to enable the feature

    # app/config/config.yml
    fos_rest:
        disable_csrf_role: ROLE_API

We have now to modify the functional tests adding the authentication: 
    
    // /src/Acme/BlogBundle/Tests/Controller/PageControllerTest.php
    public function setUp()
    {
        $this->auth = array(
            'PHP_AUTH_USER' => 'user',
            'PHP_AUTH_PW'   => 'userpass',
        );

        $this->client = static::createClient(array(), $this->auth);
    }

## Different points of view

It is very common when the API should serves different fields according to the role of the user that requests a resource.

For example the field `state` that determines whether the page is public or not, can only be viewed and changed by the administrator.
There are also more complex scenario, eg. the same field may follow different validations.

How to handle those differences on the same Page Entity?

There are many ways: validation groups, using serialization groups, or use different forms.

### The form is the resource

I personally prefer the form, an idea could be associate a form to a role. 
Given the same URI, if the user was authenticated as an administrator would use the `PageAdminType` form, otherwise she/he would use the `PageUserType`.

But this idea of associating a form to a role, does not fully comply with the REST philosophy,
 each request should be made to the desired resource, but technically the forms are different.

The URI should indicate which resource you want, and the layer of authorization should only accept or reject your request.

In the article part2, we have already seen that the resource we receive in input is not the entity Page, but it is the `pageType` form, the API uses as interface the `pageType` form.

Maybe the best option is use different URIs for different form and role.

1. `api/v1/pages/`, the form is the standard `PageType`.

2. `api/v1/admin/pages/` the form used will have more fields and different authentication.

We just apply the definition of Resource not to the Entity but to the form.

## The perfection?

In the enterprise the perfect software is when timing, clean-code and maintenance are in the correct balance with the specifications.

In the open-source and in the Symfony ecosystem developers tend to do their best,
it would be possible to improve our application and our bundle?

### Thin Bundle, Fat library.

We could improve the loose coupling leaving in the bundle only the files that are truly related to Symfony
and create a library for the entities, forms and services. But this could be another topic a blog post :).

## Recap

We are in the end of this epic tale, this recapitulate should contain few concepts:

1. We have created a smart service `PageHandler` where all the logic about the page is.

2. The service `PageHandler` is an API for all the Symfony application,
it respects the Interface `PageHandlerInterface`, and is available in the container.

3. We have developed a controller `PageController`, the actions have @docblock annotations in order to decorate the View,
those annotations also create the documentation, and provide a parameters filtering.

4. The form is very important in this REST application, the form is the input for our PUT/PATCH/POST methods,
it also validates and hydrates the object.

5. The form could also be the resource named by the URI.

6. We have defined what is the meaning of `idempotent` and why `PUT` should also create the resource.

There are a lot of concepts about REST that I won't cover here like RESTFul, HAL, OAuth ...

The [repository](https://github.com/liuggio/symfony2-rest-api-the-best-2013-way) contains all the functions and the tests that we have seen during those articles.

##### Have a REST.

That is all for the 2013

### Resources

[RFC1945 Request-URI](http://tools.ietf.org/html/rfc1945#page-24)

[RFC5988 Web Linking](http://tools.ietf.org/html/rfc5988#page-6)

[REST API at github.com/v3/](http://developer.github.com/v3/)

[gimler/symfony rest edition](https://github.com/gimler/symfony-rest-edition)

[Symfony Faking the Method](http://symfony.com/doc/current/cookbook/routing/method_parameters.html#faking-the-method-with-method)

[should-restful-apis-include-relationships/](http://idbentley.com/blog/2013/03/14/should-restful-apis-include-relationships/)

[best-practices-for-a-pragmatic-restful-api](http://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api)