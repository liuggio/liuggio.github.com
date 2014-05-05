---
layout: post
title: "Domain Driven Design principles with Symfony2 also for dummy applications"
description: "The way we develop evolves"
category: tutorial
published: true
tags: [symfony, php, bdd, ddd]
edit-link: https://github.com/liuggio/liuggio.github.com/blob/master/_posts/2014-04-29-domain-driven-design-and-symfony-for-simple-app.md
---
{% include JB/setup %}

### Part 1 - The new flow, the layered architecture and the entities.

The objective of this series of articles is to show how and why I have changed the way of programming,
I would also point to the benefits of adopting certain principles of Domain Driven Design also with simple applications, instead using DDD strictly might benefit only for some particular and complex Domains.

I do not want to say I am an evangelist of the Domain Driven Development philosophy, but the [red book of V.Vaughn](http://www.amazon.it/Implementing-Domain-Driven-Design-Vaughn-Vernon/dp/0321834577) and the movement, has profoundly changed and improved the way I code today.

I'm sorry for this new trilogy :), I would like to be short and concise bringing practical examples, as always I love to say that I didn't invent anything.

## History

Around 2003/2004 I gave birth to a PHP framework that would help me on some small software, 
was a jumble of libraries stacked with copy and paste, starting from [php-nuke](https://www.phpnuke.org/).
I released it on [sourceforge](https://www.sourceforge.net) the project had many "include" and a few contributors,
the name of the project is a secret, it was a youth error.

That was the first time I understood that the decoupling not occurs only at the class level, and the benefit of using a library for the template and a library for the database abstraction gave to me a new mindset, in the meanwhile Eric Evans was publishing [the Blue Book](http://www.amazon.it/gp/product/0321125215/ref=pd_lpo_sbs_dp_ss_1) .

My programming style has changed,
and I do not understand how music artists or painters keep their style for an entire career,
even in the same version of Symfony my developing style changes and evolves.

To better understand how changed let's start with a real example:

Stories:

1. As Author I want to be able to write a new Blog-Post as draft.
2. Given a blog-post already written, as Author I want to be able to publish a new Blog-Post.
3. The Blog-Post is written by an Author with a title and a body.
4. When a blog post is written is not public.
5. As Author I want to see a list of my posts.

### Before: code first (it means bundle-first).

If few years ago I'd thought to the  database first, 
Symfony2 gave a breath of fresh pushing forward the mentality of modularity and decoupling.

It was explicit the good practice of not putting the logic inside the controllers, but as a side effect in the ecosystem
has become quite common the "**bundle-everything with thin controllers and fat services**".

Services full of logic, and empty ORM entities (empty means only with setters and getters).

### Thin Controllers and Fat Services = smell

With a new project you start installing the symfony-standard,
you create a bundle for each concept,
then you create thousands of Doctrine2 entities (get and set on each property), and then you start testing the service before coding, then
a slim controller and a functional test over the Reponse playing with HTML and css path in order to have green tests.

This way of doing it is not even too wrong, it does not suck, but adding new features you'd have a lot of interdependent Symfony bundles, many fat services (maybe) with abstract nouns such as 'manager', 'service', 'handler' and even a CoreBundle with the shared logic.
Entities with getters, setters, but also full of annotations (doctrine and serializer).

The folders would have this structure:

	Symfony standard
	├── BlogBundle
	│    ├── Entity
	│    │   └── Blog.php
	│    │   └── BlogRepository.php
	│    ├── Service
	│    │   └── BlogService.php
	│    │   └── BlogManager.php
	│    │   └── BlogManipulator.php
	│    │   └── BlogFactory.php
	│    │   └── BlogHandler.php
	│    └── Controller
	│        └── BlogController.php
	│    CoreBundle
	│    ├── Entity
	│    │   └── Currency.php
	│    ├── Service
		 
	...


http://blog.codinghorror.com/i-shall-call-it-somethingmanager/

#### Something wrong and Anemic Domain Model

A couple of years ago after a big project in Symfony2, I realized there was something that could be improved, too many bundles, too many entities persisted without a real value, too many connections, too many fat services, too much Symfony2-centric application, something had changed, and in the my PHP community began to appear articles from [williamdurand](http://williamdurand.fr) and [Verraes](http://verraes.net/) on how we could improve this pattern.

Connected bundles wasn't the biggest problem, the system was suffering also of [Anemic Domain Model](http://www.martinfowler.com/bliki/AnemicDomainModel.html).

What is the problem of moving a lot of logic in the services?

A service should reside in the application layer should be small by definition:

<blockquote>In general, the more behavior you find in the services, the more likely you are to be robbing yourself of the benefits of a domain model. If all your logic is in services, you've robbed yourself blind.<small>Martin Fowler</small></blockquote>

## Today

Today I develop in a different way, many more [Value Objects](/persist-the-money-doctrine-value-object/), Entities with more logic, I would start by specifying use cases, and above all the flow of development has changed.

Thinking first about objects that have to perform behaviors, protagonists of the use cases are domain entities (should not be confused with the doctrine-entity) and aggregates, the step of deciding how to persist should be sent to a later stage.

### Layered Architecture

The thing that mostly changed the way I develop is the separation of the various concerns of our application by isolating the expression of domain model and business logic, eliminate any dependency from the business logic.

For [Buschmann](http://www.amazon.com/Pattern-Oriented-Software-Architecture-Volume-Patterns/dp/0471958697/ref=tmm_hrd_title_0?) there are 4 layers:

1. **Domain Layer**: the internal layer is where you have to define the outlines of the use cases
2. **Application Layer**: the flow of the behavior 
3. **User Interface layer**: how to present to the user
4. **Infrastructure Layer**: and finally how to implement them, the choice of the framework should be made in this final layer.

At the beginning we do not want to think about how we will save our objects, we just want the use cases satisfied, we do not want to think about which framework we will use, we are in the `Domain Layer`.

<blockquote>It doesn't depend on any particular UI or service interface-related technology, it just describes use cases.</blockquote>

Let's see specifically how to proceed in the development, starting from the use cases.

Install a BDD tool that helps you to write user stories:

	composer init
	composer behat/behat "2.4.*@stable"
    bin/behat --init

So we start writing the business goal in `features/post.feature`
	
	 Scenario: Write a new blog Post
	    Given I am an Author
	    When I fill in the following:
	      |  First Post | Great Description |
	    Then the "First Post" post should be written.

	  Scenario: Publish a blog Post
	    Given I am an Author
	    And I wrote a post with "First Post" and "Great Description" as description
	    When I publish the post with the title "First Post"
	    Then the "First Post" post should be public.

	  Scenario: List of all my posts
	    Given I am an Author
	    And I wrote the following posts:
	      |  First Post | Great Description |
	      |  Second Post | Bad Description |
	    When I see the list of all my posts
	    Then the "First Post" and the "Second Post" should be shown.

So we have completed the outside specification, now we could go deep inside with `phpspec` describing the behaviours of our domain.

We have recognized 2 entities `Post` and `Author` they didn't change together so they are 2 different concepts, 
they are `Entity` and not value objects because they could change, and they have a lifecycle, and we do want to identify an Author or a Post.

Install the tool that helps you to design focusing on the behaviors:

    composer require "phpspec/phpspec" "~2.0"
    bin/phpspec describe Liuggio/Blog/Post

### Post behaviours

We want to describe how the `Post` entity will works before writing the code:

##### The Blog-Post is written by an Author with a title and a body.

describing the specification in `spec/postSpec.php`:

	function it_should_be_written_by_an_author_containing_a_title_and_body()
    {
    }

    function it_should_be_written_as_drafted()
    {
    }

    function it_should_be_published()
    {
    }

Running phpspec you will have:

	$ bin/phpspec  run --format=pretty
	  post
	10  - should be written by an author containing a title and a body
	    todo: write pending example
	14  - should be written as drafted
	    todo: write pending example
	18  - should be published
	    todo: write pending example
	1 specs
	3 examples (3 pending)
	2ms

What is obvious from the first specification? we need an `Author` object.

    bin/phpspec describe Liuggio/Blog/Author

and then in `spec/Liuggio/Blog/AuthorSpec.php`
   
    function it_should_always_have_a_nickname($nickname)
    {
        $this->beConstructedWith($nickname);
        $this->shouldHaveType('Liuggio\Blog\Author');
    }

Running `bin/phpspec  run --format=pretty`
	
We are going to create the BlogAuthor:

	class Author
	{
	    private $nickname;

	    function __construct($nickname)
	    {
	        $this->nickname = $nickname;
	    }

We prefer to have in the constructor `$nickname`, in this way we are protecting the Entity `Author` to be consistent by specification (nickname could also be a Value Object).

Back to our Blog specifications we could add here the Author `spec/Liuggio/Blog/PostSpec.php`

	function it_should_be_written_by_an_author_containing_a_title_and_body(Author $author)
    {
        $this->beConstructedWith($author, 'title', 'body');
        $this->shouldHaveType('Liuggio\Blog\Post');
    }

Then we write our Post Entity Class.

	class Post
	{
	    private $author;
	    private $text;
	    private $body;

	    function __construct(Author $author, $text, $body)
	    ...

What changed is that we don't care here about Doctrine and Identity, we care only about the behaviour.

##### When a blog post is written is not public

We have to add an object that represents the state of the post, we are going to use a Value Object.

Although there are several libraries to work with Finite State Machine, maybe a simple value object in this case may be perfect.

As always, we start from the specifications to get an object that represents the states of Drafted and Published.

`spec/Liuggio/Blog/PostSpec.php`


	function it_should_be_written_as_drafted()
	{
	    $this->isPublic()->shouldBe(false);
	}

We are describing the behavior we expect the entity Post,
and we put this rule in the constructor into `\Liuggio\Blog\Post`:

	class Post
	{
		// ..
	    public function __construct(Author $author, $title, $body)
	    {
	        $this->author = $author;
	        $this->title = $title;
	        $this->body = $body;
	        $this->state = State::draft();
	        $this->createdAt = new \Datetime("now");
	    }


##### The Blog-Post could be published.

`spec/Liuggio/Blog/PostSpec.php`

    function it_should_be_published()
    {
        $this->publish();
        $this->isPublic()->shouldBe(true);
    }

    function it_should_raise_exception_during_publish_if_it_was_already_public()
    {
        $this->publish();
        $this->shouldThrow('Exception')->duringPublish();
    }

We have reached a point where our entity `Post` will be like:

	class Post
	{
	    //...

	    public function isPublic()
	    {
	        return $this->state->isPublic();
	    }

	    public function publish()
	    {
	        if (!$this->couldBePublished()) {
	            throw new \Exception("Could not do this transition");
	        }

	        return $this->state = State::published();
	    }

	    private function couldBePublished()
	    {
	        return !$this->isPublic();
	    }

The folder is now:

	My-Domain-Driven-Blog
		├── spec
		│   └── Liuggio
		│       └── Blog
		├── src
		│   └── Blog
		│       ├── Author.php
		│       ├── Post.php
		│       └── State.php

## The github repository

There's a repository at [liuggio/DDD-dummy-blog-with-symfony2](https://github.com/liuggio/DDD-dummy-blog-with-symfony2/) you could see the working code using the tag `part-1` with:

    git clone liuggio/DDD-dummy-blog-with-symfony2
    cd DDD-dummy-blog-with-symfony2
    composer install
    git checkout -f part-1

All the tags for the demo project at [tags](https://github.com/liuggio/DDD-dummy-blog-with-symfony2/tags)

### EDITED 30/04/2014

I love feedbacks:

<blockquote>Good post but Ubiquitous Language is lacking. You don't "create" blog post, it is written. A post doesn't "have" an author etc
<small><a href="https://twitter.com/mathiasverraes"><img src="http://www.gravatar.com/avatar/3a7cb0c3c8d5864e2e72c49cafc3e4d5?s=25" alt="@mathiasverraes">@mathiasverraes</a></small></blockquote>
then I modified the stories and the behaviours.

## Next?

We focused on the behaviors of the `Post` and `Author` entities during the specification,
our `spec` are green now, but our `behat` stories are red.

In the next post, we will finish adding **services** using command patterns, we will describe how to create **repositories**, how to complete the project, how to create the **business expectations over the application layer** and not on the user interface,
and how to works with Symfony2 in the **Infrastructure layer**.

Stay tuned and tweet if you want the next episode.
