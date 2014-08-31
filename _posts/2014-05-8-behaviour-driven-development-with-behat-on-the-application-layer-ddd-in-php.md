---
layout: post
title: "BDD, DDD and PHP. Behat on the Application layer"
description: "The way we develop evolves"
category: tutorial
published: false
tags: [symfony2, PHP, BDD, DDD]
edit-link: https://github.com/liuggio/liuggio.github.com/blob/master/_posts/2014-05-30-behaviour-driven-development-with-behat-on-the-application-layer-ddd-in-php
---
{% include JB/setup %}

This is the second part of a series of articles, in the first part we talked about how the design flow has changed using [a layered architecture in php](/domain-driven-design-and-symfony-for-simple-app), we started from the use cases, we created our business Entities, we wrote user stories, and we described the behaviour of the `Post` and the `Author` Entities.

We are trying to apply some DDD principles using a simple domain, because POTREMMO avere unvantaggio .... e poi

also because when a project starts you don't know yet whether it will be a complex domain, this approach is called DDD-lite.

## Domain Layer and User stories on the Application Layer.

### Remember the Domain model

We designed the use cases, we used the business language, and we have created models where all the logic lives.
The models could also contain their validators.
We gave importance to Value objects, we have created only the methods that we needed, we have avoided writing getters and setters of our Entities.

We have also protected the entities putting the needs directly in the constructor. 

**Is it time to talk about implementation?** Not now.

## Services and Use Cases

We want to satisfy the Behat features visible at `features/post.feature`

In order to do this we are going to write a class for each use case.

In our feature file we have 3 scenarios:

	Scenario: Write a new blog Post
	 ... 
	Scenario: Publish a blog Post
	 ...
	Scenario: List of all my posts
	 ...

In order to satisfy those 3 use stories, we could create 3 services that explicit and satisfy those functionalities.

The word `service` does not describe the behaviors and not explicit their functionalities,
in this case the services are use cases and they will need the infrastructure layer.

Under the namespace `Liuggio\Blog\UseCase` we are going to create 3 classes
each class get its name from a scenario:

1. WriteBlogPost
2. GetAuthorPosts
3. PublishBlogPost

Before writing the classes (as always) we are going to write the specification see `
spec/Liuggio/Blog/UseCase`, because this helps to better understand what we need and what we are going to create.

	// spec/Liuggio/Blog/UseCase/CreatePostSpec.php
	class CreatePostSpec extends ObjectBehavior
	{
	    // ...
	    function it_saves_a_post_to_the_repository(Post $post, $repository)
	    {
	        $repository->save($post)->shouldBeCalled();
	        $this->createPost($post);
	    }

the second use case produces:
	
	// spec/Liuggio/Blog/UseCase/GetAuthorPostsSpec.php
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

Those services should be thin, and they should represent the use cases using a repository to store and fetch the Blog Posts.
We said that the service should move the model's states.

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

Now we have only to create one RepositoryInterface, maybe two, one for reading and one for writing, we will not talk about CQRS, maybe for this simple Blog is not needed, but I always prefer to separate the interface and then think about whether to implement them together.

A repository:
<blockquote>
Mediates between the domain and data mapping layers using a collection-like interface for accessing domain objects.
<small><a href="http://martinfowler.com/eaaCatalog/repository.html">martinfowler/repository</a></small>
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


### Merging Repository and use cases?

The repositories should have no logic other than execute commands to or from the data mapping.

You might think that separating Use Case and Repository is an over-engineering,
but joining them there are two possible scenarios, or the repository also will absorbs all the responsibilities of the use case model (should care about moving model between states), or the service should know about the data mapping.
You might think that combining these two responsibilities leading to the disadvantage of not being able to easily change the type of storage,
well yes but the real problem is the flow, in the discovery phase maybe you do not want or you don't need to think about the implementation details.

Riprendo @couac:
<blockquote>
	In DDD, we don't consider any databases. DDD is all about the domain, not about the database, and Persistence Ignorance (PI) is a very important aspect of DDD.
<br>...<br>
	With Persistence Ignorance, we try and eliminate all knowledge from our business objects of how, where, why or even if they will be stored somewhere. <br>Persistence Ignorance means that the business logic itself doesn't know about persistence. In other words, your Entities should not be tied to any persistence layer or framework.<br>
<small><a href="http://williamdurand.fr/2013/08/20/ddd-with-symfony2-making-things-clear/">williamdurand/making-things-clear</a></small>
</blockquote>

Se vengono creati repository con della logica o delle operazioni, stiamo dando troppa responsabilità al repository, che dovrebbe essere solo la terra di mezzo, tra i dati e lo storage.

## You are here: \_ \_ \_

So we have created Entities, we have created 3 Use case Services, and we have created 2 repository interfaces, Abbiamo finito il Dominio della nostra applicazione, 
per completare le feature di behat dobbiamo solo implementare le interfacce.

Implementare? 

Mentre lo strato di dominio serve a definire la logica lo strato di applicazione should move and handle the flow of the entity.

L'application layer dovrebbe avere meno logica possibile, dovrebbe invece guidare la logica delle entità.

Lo strato di User Interface dovrebbe mostrare e ricevere nel giusto formato gli oggetti.

Esiste uno strato definito `Infrastructure` che racchiude le scelte implementative.

Quindi:

* Domain Layer could use infrastructure objects.
* Application Layer could use Domain objects and infrastructure objects.
* User Interface could use Domain, Applications and Infrastructure objects.

Non se vedete quanto la Dependency Injection è utile in questo contesto ad esempio i nostri use case services,
usano un repository che fa parte dell'infrastructure.

In questo esempio 

- **Application Layer**(**Domain**, **Infrastructure**[Repository])
- **User Interface**(**Application Layer**[Use Case], **Infrastructure**[Form, EventDispatcher, Error, Session ... ])

## Infrastruttura

Cosa ci manca per finire le nostre storie? solo uno strato di infrastruttura sulle interfacce dei repository.

Quindi agli use cases gli serve un modo per salvare e leggere alcuni dati, potremmo creare un repository,
non vogliamo ancora scegliere se utilizzare un ODM o un ORM e non sappiamo nemmeno se utilizzare l'overhead di doctrine e una Unit Of Work,
vogliamo creare un repository semplice, una implementazione potrebbe essere:

	class BasicPersistenceRepositorySpec extends ObjectBehavior
	{
	    function it_should_retrieve_a_list_of_posts_by_author_nickname(Post $post1)
	    {
	     	// ...
	        $this->save($post1);
	        $this->getPostsByAuthorNickname('liuggio')->shouldContain($post1);
 
Andiamo a sviluppare un repository semplice basato su `Key->Value`, 
aggiungiamo una libreria `composer require "doctrine/cache" "~2"`.

	class BasicPersistenceRepository implements PostRepositoryWriterInterface, PostRepositoryReaderInterface
	{
	    private $provider;

	    public function __construct(CacheProvider $provider = null)
	    {
	        $this->provider = $provider;
	        if (null === $this->provider) {
	            $this->provider = new FilesystemCache(sys_get_temp_dir());
	        }
	    }

	    public function save(Post $post)
	    {
	        $posts = array();
	        if ($this->provider->contains($post->getAuthorNickname())) {
	            $posts = $this->provider->fetch($post->getAuthorNickname());
	        }

	        $posts[$post->getTitle()] = $post;
	        $this->provider->save($post->getAuthorNickname(), $posts);
	    }

	    public function getPostsByAuthorNickname($nickname)
	    {
	        return $this->provider->fetch($nickname);
	    }
	}  

Si potrebbe creare anche un repository senza persistenza, iniettando un `ArrayCache` o un APCCache o un RediCache.

Qualcosa di veramente semplice tipo: 

Con questo Repository le nostre storie sono complete o meglio siamo in grado di far diventare le features di Behat verdi.

	
	My-Domain-Driven-Blog
		├── features
		│   ├── bootstrap
		│   │   └── FeatureContext.php
		│   └── post.feature
		├── spec
		│   └── Liuggio
		│       └── Blog
		│           └── ...
		├── src
		│   ├── Blog
		│   │   ├── Author.php
		│   │   ├── Infrastructure
		│   │   │   └── PostRepository
		│   │   │       └── BasicPersistenceRepository.php
		│   │   ├── Post.php
		│   │   ├── PostRepositoryReaderInterface.php
		│   │   ├── PostRepositoryWriterInterface.php
		│   │   ├── State.php
		│   │   └── UseCase
		│   │       ├── GetAuthorPosts.php
		│   │       ├── PublishPost.php
		│   │       └── WritePost.php
		└── vendor
		    └── ...


### Satisfying Behat tests

Is very common to create user stories against The User Interface Layer, ma questo porta sia ad un grosso dispendio di tempo per seguire javascripts ed elementi grafici,
sopratutto è molto complesso il mantenimento, se i bisogni del business cambiano con una certa velocità l'interfaccia raddoppia questa velocità, e il mantenimento risulta a volte più dispendioso dei vantaggi di avere acceptance test sulle user interface, e a volte il business non interessa avere un button con il click 'next',

Quindi potremmo addirittura scrivere funzionalità di Behat not on the User Interface layer?
Si le feature sono scritte con l'obiettivo di soddisfare il business, e non ci serve di testare tutto, come nel caso del test funzionale.

Quindi a volte è preferibile arrivare alle user stories soddisfatte senza User interface solo con Application layer e qualche oggetto dell'infrastructure.

La definizione di Martin Fowler di [SubcutaneousTest](http://martinfowler.com/bliki/SubcutaneousTest.html).

#### The test pyramid and the ice-cream-cone

<center><a href="http://martinfowler.com/bliki/TestPyramid.html"><img src="http://martinfowler.com/bliki/images/testPyramid/pyramid.png" height="130" ></a> versus the antipattern<a href="http://watirmelon.com/2012/01/31/introducing-the-software-testing-ice-cream-cone/" target="_blank"><img src="http://watirmelon.files.wordpress.com/2012/01/softwaretestingicecreamconeantipattern.png?w=300" height="130" ></a></center>

Il blog di [Martin Fowler](http://martinfowler.com/bliki/TestPyramid.html) è esaustivo.

### Satisfying business needs 

Abbiamo comunque soddisfatto le user stories con il repository see: `features/bootstrap/FeatureContext.php`

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

features satisfyed!!!

![Behat green]({{ ASSET_PATH }}/readable-liuggio/img/2014-ddd.behat.png)


## Goal

Abbiamo portato a casa, che i test sono molto uniti con il business analisys, econ lo sviluppo, 
c'è un punto che si hanno cosi tante features.

Abbiamo repository non accoppiati in nessun modo con l'implementazione.

Tutte le dipendenze esplicite, dipendenze unidirezionali, abbiamo 

- Dipendenze devono essere unidirezionali.
- una cose del genere $this->something->getXYZ()->getABC() non è proprio pulita : ?? @todo LINK
- Tell don't ask: no getters or setter, commands, Settare le proprietà e imporre i comportamenti
- Abbiamo inoltre soddisfatto il business senza dire symfony2, senza framework.
- Abbiamo imparato che avere tante classi è gratis, ed è economico se queste non contengono logica,
ma che danno una spinta verso il codice disaccoppiato.

## Next

nel prossimo articolo implementeremo la User Interface, mettendo il form al centro dell'interfaccia esterna, 
vedremo come i punti di intersezione tra BDD TDD e DDD.

come sempre twittate se volete un bel finale :).