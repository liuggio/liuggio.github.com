---
layout: post
title: "Symfony2 REST API: the best way - 2013 - article 2"
description: "REST API tutorial on symfony2 second part"
category: tutorial
tags: [rest, api, symfony2]
published: false
---
{% include JB/setup %}

## Nelle puntate precedenti

Questo è il secondo articolo che descrive come creare una applicazione REST usando Symfony2.
Nella prima parte [Symfony2 REST API: the best way](http://welcometothebundle.com/symfony2-rest-api-the-best-2013-way)
abbiamo creato le basi costruendo l'applicazione il bundle, l'entità `Page`, un Handler che gestisce la logica e un controller rest,
per ora il controller gestisce solo una get dato un `id`.

In questa articolo andremo a completare le funzionalità di creazione, modifica, e cancellazione of a `Page`.

## La creazione Post

Dobbiamo creare una API REST che permetta la creazione di una risorsa di tipo Page, 
vogliamo che chiamando la risorsa con il metodo POST all'indirizzo `/api/v1/pages.json` 
e dando come contenuto tutta risorsa serializzata otteniamo una risposta json con un 201.
	
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

Naturalmente i test sono rossi, andiamo subito ad agire aggiungendo la funzione post nel `PageHandler`.

Stiamo per creare una funzione `post` che prende dei parametri come array contenenti tutti i campi dell'entità page,
i parametri vengono validati, e un oggetto di tipo Pagina viene prima idratato e poi viene utilizzato l'object manager per salvare i contenuti con una persist:

Il primo step è creare un test che rispetti il comportamento, e successivamente scriveremo la funzione che assomiglierà anche nel nome alla funzione del controller.


    /**
     * Create a new Page.
     *
     * @param array $parameters
     *
     * @return PageInterface
     */
    public function post(array $parameters)
    {
        $page = $this->createPage();

        return $this->processForm($page, $parameters, 'POST');
    }


La funzione processForm ha la responsabilità di creare e validare and Hydratation di un oggetto di tipo Page, con il giusto HTTP method:

Avrete sicuramente usato già `form->bind(Response)`, ma in questo caso non avete la Request ma solo i valori in array dei campi da riempire quindi invece di `bind` dovremo utilizzare `submit` e poi il check su `isValid`.

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
        $form->submit($parameters, false);
        if ($form->isValid()) {

            $page = $form->getData();
            $this->om->persist($page);
            $this->om->flush($page);

            return $page;
        }

        throw new InvalidFormException('Invalid submitted data', $form);
    }

Questa funzione ritorna un oggetto di tipo PageInterface, ma se i paramtrei non sono validi
lancia una eccezione di tipo InvalidFormException che poi potrà essere `catch`ed and handled with `$e->getForm`

The test is at `src/Acme/BlogBundle/Tests/Handler/PageHandlerTest.php:63`

### Why don't inject the request?

The real question is do we really need the request?

We don't need the request here, the main responsability of `PageHandler` class is that `handle` (creating, editing, showing) the `Page` entity, given some parameters.
In this way you could use `PageHandler` from another service, without injecting or faking a Request.

`PageHandler` will contain the API for the other services of the `service.container`.

### The Post Controller

The post controller is easy all the domain logic is demanded to the PageHandler:

    /**
     * @Annotations\View(statusCode = Codes::HTTP_BAD_REQUEST)
     */
    public function postPageAction()
    {
        try {
            $newPage = $this->container->get('acme_blog.page.handler')->post(
                    $this->container->get('request')->request->all()
            );

            $routeOptions = array(
                'id' => $newPage->getId(),
                '_format' => $this->container->get('request')->get('_format')
            );

            return $this->routeRedirectView('api_1_get_page', $routeOptions, Codes::HTTP_CREATED);

        } catch (InvalidFormException $exception) {

            return $this->view(array('errors' => $exception->getForm()), Codes::HTTP_BAD_REQUEST);
        }
    }


A new page is created by the `PageHandler`, then the helper `routeRedirectView`  add to the http header 
the Location: http://localhost:8000/api/v1/pages/47.json.

If something the invalid form application


### test the application

	curl -X POST -d '{"title":"title","body":"body"}' http://localhost:8000/api/v1/pages.json --header "Content-Type:application/json" 

it will return 

	< HTTP/1.1 201 Created
	< Host: localhost:8000
	< Location: http://localhost:8000/api/v1/pages/50.json
	< Allow: POST
	< Content-Type: application/json

This is great status code `201` resource created, and location tells to the consumers where to get this resource.
and see what's happen if we try to send bad content:

	curl -X POST -d '{"tilde":"title","bold":"body"}' http://localhost:8000/api/v1/pages.json --header "Content-Type:application/json" -v

the Response will be `400`

	< HTTP/1.1 400 Bad Request
	< Host: localhost:8000
	< Content-Type: application/json
	< 
	< {"form":{"errors":["This form should not contain extra fields."],"children":{"title":[],"body":[]}}}

