---
layout: post
title: "Thin bundles, fat libraries"
description: ""
category: tutorial
published: false
tags: []
---
{% include JB/setup %}

Nel mondo dell'open source si deve sempre cercare di fare le cose per bene,
perche stiamo creando un prodotto che dovrà essere aggiornato nel minor tempo possibile,
dovrà permettere ad altri di poterlo modificare, e apprezzare.

Da quando è su symfony2 sono stati creati molti bundle, 
bundle senza librerie.

Parlando di progetti opensource è ormai chiaro che ha senso che una libreria abbia un bundle per l'integrazione su sf2 ma che non ha senso che esista un bundle senza libreria, vuol dire che quella funzionalità funzioni solo con sf2, che non ha senso estrarre le funzionalità dal framework.

Ci sono vari problemi che emergono, cosa succedere se il sito cambia?

Alcuni problematiche sono emersi da orso e dal suo talk [sf2-wtf](http://www.slideshare.net/MicheleOrselli/sf2-wtf).

Cosa succede se esce symfony3 convertiresti i tuoi progetti?

Quanto è facile programmare in modo che tutte le funzionalità siano fuori dal bundle?

ad esempio 
	
	# creiamo il progetto symfony
	composer create-project symfony/framework-standard-edition thin-bundle dev-master -sdev
	cd thin-bundle
	app/console generate:bundle --namespace=Acme/SpaceBundle --dir=src --no-interaction
	php app/console doctrine:generate:entity \
	    --entity=AcmeSpaceBundle:Star --format=annotation \
	    --fields="name:string(255) description:text" \
		--no-interaction



	src/Acme
	└── Space
	    └── SpaceBundle
	        ├── AcmeSpaceBundle.php         # - contains sf2 domain specific
	        ├── DefaultController 		    # - contains sf2 domain specific
	        │   └── DefaultController.php   # - contains sf2 domain specific
	        ├── DependencyInjection         # - contains sf2 domain specific
	        ├── Entity                      # + contains doctrine domain specific
	        │   └── Star.php                # + contains doctrine domain specific
	        │   └── StarRepository.php      # + contains doctrine domain specific
	        ├── Model                       # + 
	        │   └── StarInterface.php       # + 
	        ├── Manager                     # + 
	        │   └── StarManager.php         # + 
	        ├── Resources                   # - contains sf2 domain specific
	        │   ├── config                  # - contains sf2 domain specific
	        │   │   └── services.xml        # - contains sf2 domain specific
	        │   └── views                   # +/- could contains sf2 domain specific
	        └── Tests					    # +/- could contains sf2 domain specific
	            └── Controller              # +/- could contains sf2 domain specific
	                └── DefaultControllerTest.php # +/- could contains sf2 domain specific




