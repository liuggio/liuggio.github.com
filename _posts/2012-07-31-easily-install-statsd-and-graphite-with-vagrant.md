---
layout: post
title: "Easily install statsd and graphite with vagrant"
description: "Monitor and react: monitorize your life env"
category: tutorial
tags: [monitoring, statsd, vagrant, graphite, PHP]
published: true
edit-link: https://github.com/liuggio/liuggio.github.com/edit/master/_posts/2012-07-31-easily-install-statsd-and-graphite-with-vagrant.md
---

{% include JB/setup %}


If Engineering at Tvision has a religion, it’s the Church of Graphs. If it moves, we track it. 

Ops I already heard this sentence, please read carefully this blog post [measure-anything-measure-everything](http://codeascraft.etsy.com/2011/02/15/measure-anything-measure-everything/)

For your production env sure you need statsd, you do will want to use it.

If your salary is paid by a website, you need to **monitor and react**, you need a monitor that pushes the informations.



## Install Statsd + Graphite + Carbon + Whisper + LinuxOS + apache + python + django + mod_wsgi ...


The first time I spent 3 hours installing Graphite, 
now with vagrant you could try Statsd+Graphite in few minutes.

1. Install vagrant

   
   `gem install vagrant`  (if you need see the official documentation)

2. Installing the world with [vagrant](http://vagrantup.com/) is so easy



	`git clone https://github.com/liuggio/vagrant-statsd-graphite-puppet.git`

	`cd vagrant-statsd-graphite-puppet.git`
	
	`vagrant up`
   

   *My repo is just a fork of `Jimdo/vagrant-statsd-graphite-puppet` with a small bug fix*

3. Say 'WOOOW' then connect to 

    graphite: http://localhost:8080/
    
    statsd: 8125:udp


Everything is done, your virtual box with StatsD is ready to use, 
if you are a developer and you like web application YOU MUST use a web framework,
if you are a php developer you SHOULD use Symfony2,
if you use Symfony2 you should have a look to 
[symfony2 liuggio StatsDClientBundle](https://github.com/liuggio/StatsDClientBundle)
