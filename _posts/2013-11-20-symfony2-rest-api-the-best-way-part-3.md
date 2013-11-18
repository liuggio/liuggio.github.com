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

## CRUD world

<blockquote>
The key principles of REST involve separating your API into logical resources. These resources are manipulated using HTTP requests where the method (GET, POST, PUT, PATCH, DELETE) has specific meaning.
</blockquote>
<blockquote>
The great thing about REST is that you're leveraging existing HTTP methods to implement significant functionality on just a single /tickets endpoint. There are no method naming conventions to follow and the URL structure is clean and clear.
</blockquote>


patch è importante per modificare ad esempio lo stato di una risorsa:

o ...


Ad esempio come aggiungereste una star (favourite) to a page

story: via api is possible to star a page.

### github's star

Github api/v3 uses:

 - PUT    /gists/:id/star - to 'star' a gist
 - DELETE /gists/:id/star - to 'unstar' a gist
 - GET    /gists/:id/star - to check if a gist is starred was starred and return a 404 if it wasn't.

## Update and partial update

Updating a resource means with the `PUT` methods replace its content, instead using `PATCH` methods you could replace only the proprierties that are into the request.

We are talking about editing resources or collection of resources, not creating. 

### PUT <-> PATCH

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

The functions need as input an existing $page object and parameters, the form will do the magic.

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

We will try to reproduce all the page lifecycle putting the value of the result of the post
into a bash variable, so we could reuse. 

	location=`curl -X POST -d '{"title":"liuggio","body":"homepage"}' \
	   http://localhost:8000/api/v1/pages.json \
	   --header "Content-Type:application/json" -v 2>&1 | grep Location | cut -d \  -f 3`;

	echo "created result at: "$location

the result is

	created result at http://localhost:8000/api/v1/pages/143.json

and we are going to use this $location variable and modify the resource:

	curl -X PUT -d '{"title":"my","body":"homepage"}' $location --header "Content-Type:application/json" -v

	curl -X PATCH -d '{"body":"life"}' $location --header "Content-Type:application/json" -v

and then the get:

	curl $location --header "Content-Type:application/json" 

the result will be:

	{"id":140,"title":"my","body":"life"}

## Delete a Page

This is an exercise for you.

## get all the Pages

    /**
     * List all notes.
     *
     * @ApiDoc(
     *   resource = true,
     *   statusCodes = {
     *     200 = "Returned when successful"
     *   }
     * )
     *
     * @Annotations\QueryParam(name="offset", requirements="\d+", nullable=true, description="Offset from which to start listing notes.")
     * @Annotations\QueryParam(name="limit", requirements="\d+", default="5", description="How many notes to return.")
     *
     * @Annotations\View()
     *
     * @param Request               $request      the request object
     * @param ParamFetcherInterface $paramFetcher param fetcher service
     *
     * @return array
     */
    public function getNotesAction(Request $request, ParamFetcherInterface $paramFetcher)
    {
        $session = $request->getSession();

        $offset = $paramFetcher->get('offset');
        $start = null == $offset ? 0 : $offset + 1;
        $limit = $paramFetcher->get('limit');

        $notes = $session->get(self::SESSION_CONTEXT_NOTE, array());
        $notes = array_slice($notes, $start, $limit, true);

        return new NoteCollection($notes, $offset, $limit);
    }



###  Pagination

Envelope loving APIs typically include pagination data in the envelope itself. And I don't blame them - until recently, there weren't many better options. The right way to include pagination details today is using the Link header introduced by RFC 5988.

An API that uses the Link header can return a set of ready-made links so the API consumer doesn't have to construct links themselves. This is especially important when pagination is cursor based. Here is an example of a Link header used properly, grabbed from GitHub's documentation:

Link: <https://api.github.com/user/repos?page=3&per_page=100>; rel="next", <https://api.github.com/user/repos?page=50&per_page=100>; rel="last"
But this isn't a complete solution as many APIs do like to return the additional pagination information, like a count of the total number of available results. An API that requires sending a count can use a custom HTTP header like X-Total-Count.


### What to do with the properties that have changed like createdAt?



### Relationship

...


### Different usage for different user

Come devo fare se l'admin vede e puo' modificare alcuni campi che l'utente non puo' modificare


ad esempio per la nostra pagina



### Cleaning

Immaginando di dover pubblicare la nostra applicazione anzi il nostro bundle affiche altri utenti possano installando il bundle utilizzare la nostra applicazione dovremmo poter dar loro la possibilità di usare la propria entità e il il proprio form.

Ma perche il bundle?
Non potrebbe essere disaccoppiata la logica della nostra applicazione dal concetto di Symfony2?

Dato che l'interfaccia è il form, magari vorremmo distribuire il form e l'entità come libreria.

Sono veramente necessarie tutta questa logica nel bundle?
Se fossimo veramente fissati con il fatto del disaccoppiare riusciremmo a creare una libreria diversa dal bundle?





### Resources

[REST API at github.com/v3/](http://developer.github.com/v3/)

http://idbentley.com/blog/2013/03/14/should-restful-apis-include-relationships/

http://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api

http://tools.ietf.org/html/rfc5988#page-6