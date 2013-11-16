---
layout: post
title: "Symfony2 assets on RackSpace Cloud files"
description: ""
category: "tutorial"
tags: [rackspace, symfony2, bundle, assetic, cdn, php, assets]
published: true
---
{% include JB/setup %}

How to use assetic on rackspace with symfony2?

The problem is how to move the static files to the cloud files and get them from twig?

Following some tips from the forum [Registering (s3) stream wrapper for AsseticBundle – Symfony2 | Google Gruppi](http://groups.google.com/group/symfony2/browse_thread/thread/8e14c145683981d4)

I created this bundle  [RackspaceCloudFilesBundle](https://github.com/liuggio/RackspaceCloudFilesBundle) that handles the static files easily

## Installing

### Step 1

follow the README into the [RackspaceCloudFilesBundle](https://github.com/liuggio/RackspaceCloudFilesBundle
)

### Step 2 

modify the app/config/config.yml and the  app/parameters.ini

<script src="https://gist.github.com/2420800.js"> </script>

### Step 3 

when you will deploy a new image just run app/console assetic:dump –env=prod

**That’s it!!!!**

PLEASE FELL FREE TO CLONE AND SEND PULL REQUEST!

