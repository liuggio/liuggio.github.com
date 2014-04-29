---
layout: post
title: "part 2 - Behaviour Driven Development with Behat on the Application layer: DDD in PHP"
description: "The way we develop evolves"
category: tutorial
published: false
tags: [symfony, php, bdd, ddd]
---
{% include JB/setup %}

### Part 2 - Domain Layer and User stories on the Application Layer.

This is the second part of a series of articles, in the first part we saw how the design flow has changed using [a layered architecture](/domain-driven-design-and-symfony-for-simple-app), we started from the use cases, we create our business Entities, we wrote user stories, and we describe the behaviour of the `Post` and the `Author` Entities.

## Features

We want to satisfy the Behat features visible at `features/post.feature`

In order to do that we need to write a class for each use case.

In our feature file we have 3 features:

	Scenario: Create a new blog post
	Given I am an author
	When I fill in the following:
	  |  First Post | Great Description |
	Then the "First Post" post should be saved.

	Scenario: Publish a blog post
	Given I am an author
	And I have a post with "First Post" and "Great Description" as description
	When I publish the "First Post" post
	Then the "First Post" post should be public.

	Scenario: Get Author Posts
	Given I am an Author
	And I have written the following:
	  |  First Post | Great Description |
	  |  Second Post | Bad Description |
	When I see the list of all my post
	Then the "First Post" and the "Second Post" should be shown.

We have to create 3 use cases, we could create 3 services that explicit and satisfy those functionalities.

In the `Liuggio\Blog\Service` namespace we are going to create 3 classes
each service get the name from its scenario:

1. CreateBlogPost
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

Those services should be thin, and they should represent the use cases using a repository to store and fetch the Blog Posts.

## Repositories

Now we have only to create one RepositoryInterface, maybe two one for reading and one for writing.

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

The repository should have no logic other than execute commands to or from the data mapping, if we put some operations, if we wanted to change the repository from a memory repository very useful for tests or dev envirment to a doctrine, we should duplicate those functions.
 
## You are here: Domain Layer.

So we have created Entities, we have created 2 Use case Services, and we have created 2 repository interfaces, Abbiamo finito il Dominio della nostra applicazione, 
per completare le feature di behat dobbiamo solo implementare le interfacce.

Implementare? 

Facciamo un attimo un punto abbiamo complatato tutti gli use case, quindi abbiamo completato il Domain layer, lo strato successivo è l'application layer is where the services are, the service should move and handle the flow of the entity.

L'application layer dovrebbe avere meno logica possibile, dovrebbe invece guidare la logica delle entità.
Lo strato di User Interface dovrebbe mostrare e ricevere nel giusto formato gli oggetti.
Esiste uno strato definito `Infrastructure`  che racchiude le scelte implementative.

Quindi:

* Domain Layer could use infrastructure objects.
* Application Layer could use Domain objects and infrastructure objects.
* User Interface could use Domain Applications and Infrastructure objects.


### Behat not on the User Interface layer?!

Is very common to create user stories against The User Interface Layer, ma questo porta sia ad un grosso dispendio di tempo per seguire javascripts ed elementi grafici,
sopratutto è molto complesso il mantenimento, se i bisogni del business cambiano con una certa velocità l'interfaccia raddoppia questa velocità,
e il mantenimento risulta a volte più dispendioso dei vantaggi di avere acceptance test sulle user interface.

Quindi a volte è preferibile arrivare alle user stories soddisfatte senza User interface solo con Application layer e qualche oggetto dell'infrastructure.

### Infrastructure Layer for repository

Per ogni servizio esterno o persistenza dovremmo avere una implementazione, per esempio per i repository potremmo implementarli sia con Doctrine ORM, sia con una InMemory o anche con Redis l'importante è che l'implementazione estenda le interfacce.

Facilmente potremmo creare una applicazione con symfony con

`composer create-project symfony/framework-standard-edition my-super-dummy-project ~2`

E spostate tutto il lavoro fatto finora in my-super-dummy-project.


### User Interface 

L'user Interface dovrebbbe prendere solo le responsabilità del 
 

## Simpleton










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

