---
layout: post
title: "part 2 - BDD DDD PHP, Behaviour Driven Development with Behat on the Application layer"
description: "The way we develop evolves"
category: tutorial
published: false
tags: [symfony2, PHP, BDD, DDD]
---
{% include JB/setup %}

### Part 2 - Domain Layer and User stories on the Application Layer.

This is the second part of a series of articles, in the first part we saw how the design flow has changed using [a layered architecture in php](/domain-driven-design-and-symfony-for-simple-app), we started from the use cases, we create our business Entities, we wrote user stories, and we describe the behaviour of the `Post` and the `Author` Entities.

Stiamo cercando di apprendere alcuni principi del DDD per applicarli anche in domini non complessi, il che di per se gia un po' fa frizione con gli elementi del DDD.

#### Remember the Domain model

Quindi abbiamo disegnato i casi di uso, abbiamo spostato usando un linguaggio adatto la logica dentro dei modelli dove tutta la logica vive.
I modelli possono contenere anche un livello di validazione negli oggetti che contengono.
Le entità non devono essere confuse con le doctrine entity non è detto che coincidano quando sviluppate cercate di non pensare a come persisterete i dati.
Abbiamo dato importanza a Value objects.
Abbiamo creato solo i metodi che ci servivano, abbiamo evitato di scrivere setters e getters della nostra Entità a favore di metodi con dei significati appartenenti ad un linguaggio leggibile del dominio, abbiamo spostato la logica degli stati dentro un oggetto di valore, abbiamo inoltre protetto l'entità mettendo quello che gli serviva direttamente nel costruttore.

Is it time to talk about implementation? Not now.

## Services and Features

We want to satisfy the Behat features visible at `features/post.feature`

In order to do that we need to write a class for each use case.

In our feature file we have 3 scenarios:

	Scenario: Write a new blog Post
	 ... 
	Scenario: Publish a blog Post
	 ...
	Scenario: List of all my posts
	 ...

In order to satisfy those 3 use stories, we could create 3 services that explicit and satisfy those functionalities.

In the `Liuggio\Blog\Service` namespace we are going to create 3 classes
each service get the name from its scenario:

1. WriteBlogPost
2. GetAuthorPosts
3. PublishBlogPost

Before writing the classes (as always) we are going to write the specification see `
spec/Liuggio/Blog/Service`.

	// spec/Liuggio/Blog/Service/CreatePostSpec.php
	class CreatePostSpec extends ObjectBehavior
	{
	    // ...
	    function it_saves_a_post_to_the_repository(Post $post, $repository)
	    {
	        $repository->save($post)->shouldBeCalled();
	        $this->createPost($post);
	    }

the second use case produces:
	
	// spec/Liuggio/Blog/Service/GetAuthorPostsSpec.php
    class GetAuthorPostsSpec extends ObjectBehavior
	{
	    //..
	    function it_get_the_posts_list_from_the_repository(Author $author, $repository)
	    {
	        $author->getNickname()->willReturn('liuggio');
	        $repository->getPostsByAuthorNickname('liuggio')->shouldBeCalled();
	        $this->getAuthorPosts($author);
	    }

The third specification is really similar to the first one, and I will omit here.

La parola servizio è una parola che non parla, non descrive il comportamento e non esplicità la loro funzionalità,
in questo caso i servizi sono UseCase utilizzeranno uno strato di infrastruttura.

Those services should be thin, and they should represent the use cases using a repository to store and fetch the Blog Posts.

	//src/Blog/UseCase/PublishPost.php
	class PublishPost
	{
	    private $repository;

	    function __construct(PostRepositoryWriterInterface $repository)
	    {
	        $this->repository = $repository;
	    }

	    public function publishPost(Post $post)
	    {
	        $post->publish();
	        $this->repository->save($post);
	    }
	}


## Repository

Now we have only to create one RepositoryInterface, maybe two, one for reading and one for writing, non parleremo di CQRS, o meglio per questo semplice Blog non serve, preferisco sempre scindere le interfacce e poi pensare all'implementazione se unirle.

<blockquote>
Mediates between the domain and data mapping layers using a collection-like interface for accessing domain objects.
<small>http://martinfowler.com/eaaCatalog/repository.html</small>
</blockquote>

The repository is not the same as doctrine repository (or could be but not here),
they should represent the data mapping, using the domain language.

`Blog\PostRepositoryReaderInterface`

	interface PostRepositoryReaderInterface {
	    public function getPostsByAuthorNickname($nickname);
	} 

`Blog\PostRepositoryWriterInterface`

	interface PostRepositoryWriterInterface {
	    public function save(Post $post);
	} 

The repositories should have no logic other than execute commands to or from the data mapping, if we put some operations, if we wanted to change the repository from a memory repository very useful for tests or dev envirment to a doctrine, we should duplicate those functions.
 
## You are here: lost.

So we have created Entities, we have created 3 Use case Services, and we have created 2 repository interfaces, Abbiamo finito il Dominio della nostra applicazione, 
per completare le feature di behat dobbiamo solo implementare le interfacce.

Implementare? 

Mentre lo strato di dominio serve a definire la logica lo strano di applicazione should move and handle the flow of the entity.

L'application layer dovrebbe avere meno logica possibile, dovrebbe invece guidare la logica delle entità.

Lo strato di User Interface dovrebbe mostrare e ricevere nel giusto formato gli oggetti.

Esiste uno strato definito `Infrastructure` che racchiude le scelte implementative.

Quindi:

* Domain Layer could use infrastructure objects.
* Application Layer could use Domain objects and infrastructure objects.
* User Interface could use Domain, Applications and Infrastructure objects.

Cosa ci manca per finire le nostre storie? solo uno strato di infrastruttura sulle interfacce dei repository,

Quindi agli use cases gli serve un modo per salvare e leggere alcuni dati, potremmo creare un repository,
che tiene tutto in memoria, molto utile sopratutto per l'ambiente di `dev`.

	class InMemoryPostRepository implements PostRepositoryWriterInterface, PostRepositoryReaderInterface
	{
	    public $buffer = array();

	    public function save(Post $post)
	    {
	        $this->buffer[$post->getAuthorNickname()][$post->getTitle()] = $post;
	    }

	    public function getPostsByAuthorNickname($nickname)
	    {
	        return $this->buffer[$nickname];
	    }
	}

Sarebbe molto facile implementare un RedisPostRepository, o un APCuRedisRepository, vedremo in seguito come creare un DoctrineORMRepository...

Con questo InMemoryPostRepository le nostre storie sono complete o meglio siamo in grado di far diventare le features verdi.

### Behat not on the User Interface layer?!

Is very common to create user stories against The User Interface Layer, ma questo porta sia ad un grosso dispendio di tempo per seguire javascripts ed elementi grafici,
sopratutto è molto complesso il mantenimento, se i bisogni del business cambiano con una certa velocità l'interfaccia raddoppia questa velocità,
e il mantenimento risulta a volte più dispendioso dei vantaggi di avere acceptance test sulle user interface.

Quindi a volte è preferibile arrivare alle user stories soddisfatte senza User interface solo con Application layer e qualche oggetto dell'infrastructure.

Abbiamo comunque soddisfatto le user stories si veda `features/bootstrap/FeatureContext.php`

	class FeatureContext extends BehatContext
	{
		// ...
	    /**
	     * @Then /^the "([^"]*)" post should be written\.$/
	     */
	    public function thePostShouldBeWritten($title)
	    {
	        $this->createPostUseCase->createPost($this->post);
	        $posts = $this->getAuthorPostsUseCase->getAuthorPosts($this->author);
	        ...


## When the outside of Outside-in is inside?

Forse quello che è implicito è che non appare è che si le storie sono soddisfatte da un inmemoryRepository ma che 
c'è una storia invisibile che dice che i post dovrebbero essere persisiti in maniera permanente con il passare dei giorni,
con un InMemoryRepository stiamo mentendo alle user stories le stiamo ingannando dobbiamo prepararci 

Ci manca ancora molto, per arrivare alla fine, l'user interface anche se dovrebbe essere privo di logica complessa,
è pieno di insidie :).


L'obiettivo del BDD?
L'obiettivo delle Acceptance test?
Veramente al business non gli interessa che ci sia un bottone?
Bisogna creare altre user stories es as API customer I want to ...

Ci serve un adapter per doctrine per far si di utilizzare le sue entità senza combattere il framework.


### Infrastructure Layer for repository

Per ogni servizio esterno o persistenza dovremmo avere una implementazione, per esempio per i repository potremmo implementarli sia con Doctrine ORM, sia con una InMemory o anche con Redis l'importante è che l'implementazione estenda le interfacce.

Facilmente potremmo creare una applicazione con symfony con

`composer create-project symfony/framework-standard-edition my-super-dummy-project ~2`

E spostate tutto il lavoro fatto finora in my-super-dummy-project folder merging correctly the composer.json 

testando che il lavoro funzioni (dovrebbe) eseguendo `bin/behat && bin/phpspec run --format=pretty`

Il primo obiettivo è soddisfare le storie con un doctrine Entity e doctrine Repository.

creaiamo un bundle, aggiungiamo una Entità post e il suo repository.

Che nome dare al bundle?

beh Doctrine è in realtà solo una scelta implementativa e dovremmo mettere un adapter tra il nostro dominio e doctrine per essere funzionante.

Beh dell'adapter mi serviranno sicuramente i medatada su come persistere (join etc...), e anche campi come ID etc...

Una possibile soluzione potrebbe essere:

Blog/Infrastructure/Repository/Doctrine/Entity....
Blog/Infrastructure/Repository/Doctrine/Repository...
Blog/Infrastructure/Repository/Doctrine/Metadata...

Beh questo in symfony non è cosi semplice e sarebbe anche poco leggibile da altri abituati a lavorare con symfony,
forse per "non combattere il framework" è ancora un buon consiglio forse potremmo creare un bundle
che ricorda che è un adapter per doctrine

dentro: Blog/Infrastructure/BlogDoctrineAdapterBundle ?

E poi tutti i casi di uso dell'user Interface in un altro bundle?
es: Blog/UIBundle/Controller, Blog/UI/Form, views etc?

Sicuramente Il repository in questo caso è una scelta implementativa che appartiene all'Application Layer,
e Controller etc è una scelta implementativa che appartiene All'user interface,
Ipoteticamente potremmo sostituire doctrine con un RedisRepository, e L'userInterface non dovrebbe venire a conoscienza.

Di certo ci deve essere sempre una parte razionale. che ci tiene uniti.


Cosa Abbiamo portato a casa:

- Non c'è una fine al Decoupling, l'importante è capire fino a dove si potrebbe arrivare per poi portare a razionalità in base al dominio.
- Dipendenze devono essere unidirezionali.
- una cose del genere $this->something->getXYZ()->getABC() non è proprio pulita : ?? @todo LINK
- Tell don't ask: no getters or setter, commands, Settare le proprietà e imporre i comportamenti

Iniziamo a programmare, ascoltando i nouns, sarebbe utile programmare e disegnare il codice senza focalizzarci sui 'NOUN' ma sui 'VERB'.
The verbs should follow the noun, the noun should follow the verb.
One single model is a failure.
Exists an ubiquitous language foreach subdomain, and a ub.lang has a value only in a context.
We didn't care from proprierties.
Iniziamo a unire insieme i concetti che vanno insieme, che cambiano insieme,
cerchiamo i bounderies.
Ad esempio una proprietà non è detto che deve essere attaccata alla sua entità, 
l'età di un customer non è rilevante se serve solo per fare indagini statistiche.

Product name, description, image go togheter









### Infrastructure Layer for repository

Per ogni servizio esterno o persistenza dovremmo avere una implementazione, per esempio per i repository potremmo implementarli sia con Doctrine ORM, sia con una InMemory o anche con Redis l'importante è che l'implementazione estenda le interfacce.

Facilmente potremmo creare una applicazione con symfony con

`composer create-project symfony/framework-standard-edition my-super-dummy-project ~2`

E spostate tutto il lavoro fatto finora in my-super-dummy-project.


### User Interface 

L'user Interface dovrebbbe prendere solo le responsabilità del 
 

## Simpleton






Conclusioni

Abbiamo utilizzato una mentalità comportamentale e alcuni aspetti del DDD, abiamo preso il meglio dai due mondi
ma ci siamo persi qualcosa, ci siamo persi l'iteratività che in alcuni domini complessi è uno step obbligatorio,
q

TDD and BDD = share The big cost of change is not fixing problem, is finding problem
TDD and DDD = share experimentation, Iterative design.
DDD and BDD = Value of collaboration of business process.







#### Entity

Le entità possono essere il centro del livello di dominio dove tutta la logica vive.
Le entità possono contenere anche un livello di validazione negli oggetti che contengono.
Le entità non devono essere confuse con le doctrine entity non è detto che coincidano quando sviluppate cercate di non pensare a come persisterete i dati.
Le entità contengono Value objects.
Abbiamo creato solo i metodi che ci servivano, abbiamo evitato di scrivere setters e getters della nostra Entità, abbiamo spostato la logica degli stati dentro un oggetto di valore, abbiamo inoltre protetto l'entità mettendo quello che gli serviva direttamente nel costruttore.

Daboom: Abbiamo creato il Dominio della nostra applicazione!

We want to talk about how to implement this with doctrine and symfony?

not now :)

The application layer is where the services are, the service should move and handle the flow of the entity.

L'application layer dovrebbe avere meno logica possibile, dovrebbe invece guidare la logica delle entità. 

* Domain Layer could use infrastructure objects.
* Application Layer could use Domain objects and infrastructure objects.
* User Interface could use Domain Applications and Infrastructure objects.

L'application layer di questo esempio contiene due servizi in `\Liuggio\Blog\Service\*`.

Le feature di Behat quindi possiamo scriverle contro i due servizi.
I due servizi sono molto snelli 

`src/Blog/Service/GetAuthorPosts`

    public function getAuthorPosts(Author $author)
    {
       return $this->repository->getPostsByAuthorNickname($author->getNickname());
    }

`src/Blog/Service/CreatePost`

    public function createPost(Post $post)
    {
        $this->repository->save($post);
    }

    public function publishPost(Post $post)
    {
        $post->publish();
        $this->repository->save($post);
    }

vedere nel repository github la loro descrizione `spec/Liuggio/Blog/Service/CreatePostSpec.php` and `spec/Liuggio/Blog/Service/GetAuthorPostsSpec.php`.


### Infrastructure

Per ogni servizio esterno o persistenza dovremmo avere una implementazione, per esempio per i repository potremmo implementarli sia con Doctrine ORM, sia con una InMemory o anche con Redis l'importante è che l'implementazione estenda le interfacce.


### User Interface 

L'user Interface dovrebbbe prendere solo le responsabilità del 
 

## Simpleton


1. The Blog-Post has always an Author, a title and a body.
2. When a blog post is created is not public.
3. The Blog-Post could be published.
4. An Author could write a blogpost
5. An Author could publish a blogpost
6. An Author could view the list of all the blog post he wrote.
 


Don't be stupid start from genius:

http://verraes.net/2011/10/code-folder-structure/
http://williamdurand.fr/2013/08/07/ddd-with-symfony2-folder-structure-and-code-first/
http://richardmiller.co.uk/2013/06/18/symfony2-bundle-structure-bundles-as-integration-layers-for-libraries/
https://skillsmatter.com/skillscasts/1806-talk-from-udi-dahan


~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Behavioural test, we don't care about we don't test everything we care only what we business needs.
we don't care, Functional you test anything.

Who pick what is important and not to create behaviour?
Business decides this.

Integratational test: how to integration 2 modules works.
Functional test is a test aim to test the 'face' functionality to a use case from a user perspective.
Functional is an integration test.

