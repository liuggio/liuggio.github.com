---
layout: post
title: "part 3 - Symfony2 REST API: the best way"
description: "REST API tutorial on symfony2 third part"
category: tutorial
tags: [rest, api, symfony2]
published: false
date: 2013-11-20 13:00:00
---
{% include JB/setup %}

### Part 3 - the rest

patch è importante per modificare ad esempio lo stato di una risorsa:

o ...







This is the third article: [Part 1](http://welcometothebundle.com/symfony2-rest-api-the-best-2013-way/), [Part 2](http://welcometothebundle.com/symfony2-rest-api-the-best-2013-way/).

In this part, we are going to show how to properly modify a given resource, using `PUT`,`PATCH` and deleting with the verb `DELETE`.

## The github repository

There's a repository at [liuggio/symfony2-rest-api-the-best-2013-way](https://github.com/liuggio/symfony2-rest-api-the-best-2013-way/)
you could see the working code using the tag `part3` with

    php composer.phar create-project liuggio/symfony2-rest-api-the-best-2013-way blog-rest-symfony2 -sdev
    cd blog-rest-symfony2
    git checkout -f part3
    bin/phpunit -c app

All the tags for the demo project are at [tags](https://github.com/liuggio/symfony2-rest-api-the-best-2013-way/tags), and also you could compare the 2 articles with [compare/part2...part3](https://github.com/liuggio/symfony2-rest-api-the-best-2013-way/compare/part2...part3).

## HTTP Methods

Where possible, API strives to use appropriate HTTP verbs for each action.

 - HEAD Can be issued against any resource to get just the HTTP header info.

 - GET 	Used for retrieving resources.

 - POST 	Used for creating resources, or performing custom actions (such as merging a pull request).

 - DELETE 	Used for deleting resources.

 - PATCH 	Used for updating resources with partial JSON data. For instance, an Issue resource has title and body attributes. A PATCH request may accept one or more of the attributes to update the resource. PATCH is a relatively new and uncommon HTTP verb.

 - PUT 	Used for replacing resources or collections. For PUT requests with no body attribute, be sure to set the Content-Length header to zero.

### Safe and idempotent:

**Safe methods**

<blockquote>
	In particular, the convention has been established that the GET and HEAD methods SHOULD NOT have the significance of taking an action other than retrieval. These methods ought to be considered "safe". This allows user agents to represent other methods, such as POST, PUT and DELETE, in a special way, so that the user is made aware of the fact that a possibly unsafe action is being requested. 
</blockquote>

**Idempotent methods**

<blockquote>
	Methods can also have the property of "idempotence" in that (aside from error or expiration issues) the side-effects of N > 0 identical requests is the same as for a single request. The methods GET, HEAD, PUT and DELETE share this property. Also, the methods OPTIONS and TRACE SHOULD NOT have side effects, and so are inherently idempotent. 
</blockquote>

[http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html](http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html)

## Update and partial update

Updating a resource means with the `PUT` methods replace its content, instead using `PATCH` methods you could replace only the proprierties that are into the request.

We are talking about editing resources or collection of resources, not creating. 

### PUT PATCH

Implements those methods is simple, all the job is already done.

As always we need the story and the tests.

Adding `PageHandlerInterface` the new methods
then to the `PageHandler`:

    /**
     * Edit a Page.
     *
     * @param PageInterface   $page
     * @param array           $parameters
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
     * @param PageInterface   $page
     * @param array           $parameters
     *
     * @return PageInterface
     */
    public function patch(PageInterface $page, array $parameters)
    {
        return $this->processForm($page, $parameters, 'PATCH');
    }


### So damn easy?

**yes!**

The functions want as input an existing $page object and parameters, the form will do the magic.

### The Controller

In the controller we have to handle the Status code, the headers, and the fecthing of the page object.

The test for putAction is similar to the patchAction

	public function testJsonPutPageAction()
    {
        $fixtures = array('Acme\BlogBundle\Tests\Fixtures\Entity\LoadPageData');
        $this->customSetUp($fixtures);
        $pages = LoadPageData::$pages;
        $page = array_pop($pages);

        $this->client = static::createClient();
        $this->client->request(
            'PUT',
            sprintf('/api/v1/pages/%d.json', $page->getId()),
            array(),
            array(),
            array('CONTENT_TYPE' => 'application/json'),
            '{"title":"abc","body":"def"}'
        );

        $page->setTitle('abc');
        $page->setBody('def');

        $this->assertJsonResponse($this->client->getResponse(), 204, false);

        $updatedPage = $this->getContainer()->get('acme_blog.page.handler')->get($page->getId());
        $this->assertEquals($updatedPage, $page);
    }

and the controller will be:


    /**
     * Update existing page from the submitted data or create a new page at a specific location.
     *
     * @ApiDoc(
     *   resource = true,
     *   input = "Acme\DemoBundle\Form\PageType",
     *   statusCodes = {
     *     204 = "Returned when successful",
     *     400 = "Returned when the form has errors"
     *   }
     * )
     *
     * @Annotations\View(
     *  template = "AcmeBlogBundle:Page:editPage.html.twig"
     * )
     *
     * @param int     $id      the page id
     *
     * @return FormTypeInterface|RouteRedirectView
     *
     * @throws NotFoundHttpException when page not exist
     */
    public function putPageAction($id)
    {
        try {

            $page = $this->container->get('acme_blog.page.handler')->put(
                $this->getOr404($id),
                $this->container->get('request')->request->all()
            );

            $routeOptions = array(
                'id' => $page->getId(),
                '_format' => $this->container->get('request')->get('_format')
            );

            return $this->routeRedirectView('api_1_get_page', $routeOptions, Codes::HTTP_NO_CONTENT);

        } catch (InvalidFormException $exception) {

            return array('form' => $exception->getForm());
        }
    }

    /**
     * Update existing page from the submitted data or create a new page at a specific location.
     *
     * @ApiDoc(
     *   resource = true,
     *   input = "Acme\DemoBundle\Form\PageType",
     *   statusCodes = {
     *     204 = "Returned when successful",
     *     400 = "Returned when the form has errors"
     *   }
     * )
     *
     * @Annotations\View(
     *  template = "AcmeBlogBundle:Page:editPage.html.twig"
     * )
     *
     * @param int     $id      the page id
     *
     * @return FormTypeInterface|RouteRedirectView
     *
     * @throws NotFoundHttpException when page not exist
     */
    public function patchPageAction($id)
    {
        try {

            $page = $this->container->get('acme_blog.page.handler')->patch(
                $this->getOr404($id),
                $this->container->get('request')->request->all()
            );

            $routeOptions = array(
                'id' => $page->getId(),
                '_format' => $this->container->get('request')->get('_format')
            );

            return $this->routeRedirectView('api_1_get_page', $routeOptions, Codes::HTTP_NO_CONTENT);

        } catch (InvalidFormException $exception) {

            return array('form' => $exception->getForm());
        }
    }

### The page lifecycle


	curl -X POST -d '{"title":"liuggio","body":"homepage"}' http://localhost:8000/api/v1/pages.json --header "Content-Type:application/json" -v 2>&1 | grep Location

the result is

	< Location: http://localhost:8000/api/v1/pages/140.json

and we are going to use this path and modify the resource:

	curl -X PUT -d '{"title":"my","body":"homepage"}' http://localhost:8000/api/v1/pages/140.json --header "Content-Type:application/json" -v

	curl -X PATCH -d '{"body":"life"}' http://localhost:8000/api/v1/pages/140.json --header "Content-Type:application/json" -v

and then the get:

	curl http://localhost:8000/api/v1/pages/140.json --header "Content-Type:application/json" 

the result will be:

	{"id":140,"title":"my","body":"life"}

### Resources

[REST API at github.com/v3/](http://developer.github.com/v3/)


