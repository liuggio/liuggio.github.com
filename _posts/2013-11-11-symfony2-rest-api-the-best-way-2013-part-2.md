---
layout: post
title: "Symfony2 REST API: the best way - 2013 - article 2"
description: "REST API tutorial on symfony2 second part"
category: tutorial
tags: [rest, api, symfony2]
published: false
---
{% include JB/setup %}

Questa è il secondo articolo che descrive come creare una applicazione REST usando Symfony2.
Nella prima parte [Symfony2 REST API: the best way](http://welcometothebundle.com/symfony2-rest-api-the-best-2013-way)
abbiamo creato le basi creando l'applicazione il bundle, l'entità `Page`, un Handler che gestisce la logica e un controller rest,
per ora il controller gestisce solo una get dato un `id`.

In questa articolo andremo a completare le funzionalità di creazione, modifica, e cancellazione of a `Page`.

## Post

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

Naturalmente i test sono rossi, andiamo subito ad agire aggiungendo la funzione post nell `PageHandler`.

Stiamo per creare una funzione `post` che prende dei parametri come array contenenti tutti i campi dell'entità page,
i parametri vengono validati, e un oggetto di tipo Pagina viene prima idratata e poi viene utilizzato l'object manager per salvare i contenuti.

il primo step è creare un test che rispetti il comportamento.

La funzione post($parameter), 



