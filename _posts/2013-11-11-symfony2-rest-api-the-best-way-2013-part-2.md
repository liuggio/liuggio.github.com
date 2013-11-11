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

Per prima cosa andiamo a creare la creazione

