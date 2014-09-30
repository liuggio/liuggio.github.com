---
layout: post
title: "Speed up PHPUnit and functional tests with Symfony and fastest"
description: "No more slow test suite"
category: post
published: true
tags: [phpunit, fixture, test, functional]
edit-link: https://github.com/liuggio/liuggio.github.com/blob/master/_posts/2014-09-24-speedup-symfony-functional-tests-phpunit.md
---
{% include JB/setup %}

Once in a while a 35 minutes test suite reached 7 minutes thanks to a magician called `fastest`

... today we decided to release `fastest` as open-source library.

### Just Testing

Have you ever seen how your computer's multi-core works when you execute PHPUnit? 

![CPU with Htop and PHPUnit]({{ ASSET_PATH }}/readable-liuggio/img/phpunit-single-cpu.png)

Ehm technically only one core is fatigued, because the tests are serially launched.

## Parallel testing

### Existing tool

Maybe we can do better, we can use all cores, we have begun to experiment [paratest](https://github.com/brianium/paratest), and subsequently 
[Parallel](https://github.com/grosser/parallel). 

Parallel is the one that gave us more satisfaction, but it was not enough, Parallel does not help the functional tests.

### The needs

What we needed was to run tests in parallel *limiting* the number of simultaneously tests by the number of cores on the computer (like parallel), not only by providing information of which *channel* is being used by environment variables (as paratest does) but a stable running suite with a lot of information for each test the goal is to have **to easy make functional parallel testing*.

### How?

![CPU with Htop and Fastest]({{ ASSET_PATH }}/readable-liuggio/img/fastest-4-cpu.png)

The picture is pretty self-explanatory: using that F*#@ing **Multi-Core:**

Usually functional tests have assertions on data storage, so the best is to divide tests into channels (one per Core), and each channel is associated to a database with a different name, and only one test is executed simultaneously per channel:
eg. the test in the channel number 3 will read from the "test_3" database.

#### Installing fastest

All the info to install [fastest](git@github.com:liuggio/fastest.git) if you want a short version:

add to your `composer.json`

    "require-dev": {
        "liuggio/fastest": "dev-master"
    }


Each test will have access to the following variables:

1. Unique number for each test (useful for filename fixtures).
2. Channel number (from 1 to the core number).
3. Readeable database name.
4. Variable tells if is the first test on its channel, useful for clear cache (you'll see later how this would help).

In PHP is so easy to access to the env variable using: `getenv('varname')`.

## Improving Symfony testing speed

As we said we want to let the test decide which database use, Symfony allows you to pass the [environment variables](http://symfony.com/doc/current/cookbook/configuration/external_parameters.html) as container parameters respecting the [12 factors](http://12factor.net/config) **but** Symfony freezes the container, so the stylish way to let the environment variables decide the name of the database to use is to to decorate the DBAL Connection Factory as follow:

    `config_test.yml`
    parameters:
        doctrine.dbal.connection_factory.class: Liuggio\Fastest\Doctrine\DbalConnectionFactory

When a new connection is needed the factory will provide the correct `db_name` replacing the db name with the env. variable, using only one test environment.

    // src/Doctrine/DbalConnectionFactory.php
    /**
     * Create a connection by name, replacing the db name with the env. variable
     */
    public function createConnection(array $params, Configuration $config = null, EventManager $eventManager = null, array $mappingTypes = array())
    {
        $params['dbname'] = $this->getDbNameFromEnv($params['dbname']);

        return parent::createConnection($params, $config, $eventManager, $mappingTypes);
    }
    ...
    private function getDbNameEnvValue()
    {
        return getenv(EnvCommandCreator::ENV_TEST_DB_NAME);
    }

 
To test the setting, execute 

`export ENV_TEST_DB_NAME='test_1';app/console doctrine:database:create --env=test;`

You should see something similar to:

    Created database for connection named `test_1`

## First Big Goal

I thing you are ready to run your suite with

    find src/* -name "*Test.php" |   \
      bin/fastest                    \
        --before="bin/initORM test"  \
        --verbose                    \
        --preserve-order             \
        "bin/phpunit -c app {};"

[InitORM is a simple script](https://gist.github.com/liuggio/e927a86c8fa4dadbe828) that executes `doctrine:database:create --env=test`, `doctrine:database:drop --env=test`, and `doctrine:fixture:load --env=test`...

### A lot of failures ?!

Fastest executes each test in a single process, this mode may brings up many issues that are hided executing in the whole suite, the most well known are the namespace doesn't not match with the filename, and the file given in input by the pipe match with the "*Test.php" but they might be abstract classes.

If you want you could run Fastest with the `--xml` option instead piping the find result.

If you don't use the strategy of loading the fixtures the test need before it, may happens that a test modifies the content of the data storage, and the next test on the same channel fails.
This is very annoying and adding the option `--rerun-failed` fastest will run again all the failed tests.

If you are developing a new project or your test are isolated you may be interested on removing the `--preserve-order` option.

## Don't check if the cache is fresh! 

In the second trick we will try to avoid a lot of disk accesses, using the env. variable `ENV_TEST_IS_FIRST_ON_CHANNEL` provided by fastest.

Strongly inspired by the [kriswallsmith's trick for phpunit optimization](http://kriswallsmith.net/post/27979797907/get-fast-an-easy-symfony2-phpunit-optimization), with a little modification for our parallel domain, we will save precious minutes by removing the cache is fresh check.

The trick is quite easy instead every requests the cache is checked only once for channel process.

    protected function initializeContainer()
    {   
        $isFirst = getenv(\Liuggio\Fastest\Process\EnvCommandCreator::ENV_TEST_IS_FIRST_ON_ITS_THREAD);

        if ('test' !== $this->getEnvironment()
         || null === $isFirst
         || 0 !== (int)$isFirst
        ) {
            parent::initializeContainer();
            return;
        }

        $class = $this->getContainerClass();
        $cache = new \Symfony\Component\Config\ConfigCache($this->getCacheDir().'/'.$class.'.php', $this->debug);

        require_once $cache;
        $this->container = new $class();
        this->container->set('kernel', $this);   
    }

## Moral

Maybe it's time to think about that if you have tests that take too long is not the fault of the speed of the computer, but how was the code coupled...

Enjoy parallelization.