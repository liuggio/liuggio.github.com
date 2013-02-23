---
layout: post
title: "Using Github or Bitbucket as Assets container for Symfony2"
description: ""
category: "hack"
tags: [assetic, symfony2, bundle, github, bitbucket, php, assets]
published: true
---
{% include JB/setup %}

I made the following changes to the config, deps and autoload in order to push all the assets directly into a github repository.

** Why you should pay for CDN if you can use Github? **

## Theory

1. [Php library] the library teqneers/PHP-Stream-Wrapper-for-Git  registers a stream and commit to a repository each file that is moved throw the stream

2. [Symfony2] Assetic bundle compresses and streams the files in your local repository, and twig render the raw.github url

3. [Git-Hook] The Post-Commit hook will push to remote repository automatically


## How To

Just 3 steps

## Step 1 Install the library in your symfony2

In your deps file add the following lines, and then run bin/vendors install

<script src="https://gist.github.com/2427058.js?file=deps"> </script>

Register namespace and register stream into `app/autoload.php`

‘/usr/bin/git’  is where your git binary are

‘php-git’  is the name of the stream


<script src="https://gist.github.com/2427058.js?file=app-autoload.php"> </script>



## Step 2 Github or BitBucket Repository

- Create Repo in the root /UsingGitHubAsAssetsCloudFiles

[http://help.github.com/create-a-repo/](http://help.github.com/create-a-repo/)

the public url of the repo will be  www.github.com/liuggio/UsingGitHubAsAssetsCloudFiles,

your remote name is ‘origin’  and the branch is ‘master’


- Add the Hook into the /UsingGitHubAsAssetsCloudFiles

create a file into ‘/UsingGitHubAsAssetsCloudFiles/.git/hooks/post-commit’

<script src="https://gist.github.com/2427058.js?file=UsingGitHubAsAssetsCloudFiles-.git-hooks-post-commit"> </script>

##Step 3 Config.yml and Assets

- modify your `app/config/config.yml` in 2 different places

<script src="https://gist.github.com/2427058.js?file=app-config-config.yml"> </script>


- dump all the file for prod

`app/console assetic:dump –env=prod`

go to your symfony2 website  http://yoursymfony2.com/web/app.php/hello/{YES}

THAT’S ALL.

Now github is hosting your assets (for free) and your web server is happy (and if your webserver is into the cloud you’ll not charged for assets  )


