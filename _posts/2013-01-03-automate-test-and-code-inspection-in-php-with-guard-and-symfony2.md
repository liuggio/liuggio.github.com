---
layout: post
title: "Automate Test and Code Inspection in PHP with Guard, and Symfony2"
description: "Automate Test and Code Inspection with Guard and Symfony2"
category: phpunit
tags: [phpunit, symfony2, guard, php, automate, code-inspection]
---
{% include JB/setup %}


In everyday life there are tools that can not only speed, but also lighten the workload of your mind,
in that regard I wanted to share a useful library: [Guard](https://github.com/guard/guard).

## `Watch`  Guard!

[Guard](https://github.com/guard/guard) is written in Ruby, it automates commands based on events that happen in the filesystem.

[Guard](https://github.com/guard/guard) in a short time has become popular especially in the automate testing.

You can easily edit your files, having only the front window of your editor/IDE, each change will run the test, and you will be notified.

Indeed during these years of development, especially in PHP I overstimulated shortcuts, I have become a slave of the keyboard, Alt-Tab, Key-Up + Enter for example, are a must for programmers TDD.

For years I had the tic CTRL+S, in fact in my life I have saved (for mistake) around 2000 pages browsing with Firefox.

Before finding out [Guard](https://github.com/guard/guard), I was using a crude one-line script,
it runs each 3 seconds PHPUnit if error otherwise waits 10 seconds

``` bash
while true; do clear; phpunit; if [ ! $? ]; then sleep 10; else sleep 3;fi; done;
```

but it was not enough, I needed for something which could perform only the Test for the file that I had changed: [guard-phpunit](https://github.com/Maher4Ever/guard-phpunit)

In the PHP world, [Guard](https://github.com/guard/guard) is not so popular but instead would require much more importance.

## **Install** [guard-phpunit](https://github.com/Maher4Ever/guard-phpunit)!

Simply follow the instructions of the README.md https://github.com/Maher4Ever/guard-phpunit


### 1 Install guard-phpunit

`gem install guard-phpunit`

### 2 Step creare un file  Guardfile


    guard 'phpunit', :tests_path => 'Tests', :cli => '--colors' do
     # Watch tests files
     watch(%r{^.+Test\.php$})
     # Watch library files and run their tests
     watch(%r{^Object/(.+)\.php}) { |m| "Tests/#{m[1]}Test.php" }
    end


### 3 Run


    liuggio@liuggio:/var/repos/StatsDClientBundle$ guard
    20:33:21 - INFO - Guard uses NotifySend to send notifications.
    20:33:21 - INFO - Guard uses TerminalTitle to send notifications.
    20:33:21 - INFO - Running all tests
    20:33:23 - INFO - ..................
    > [#51616B16C20E]
    > [#51616B16C20E] Finished in 2 seconds
    > [#51616B16C20E] 18 tests, 43 assertions
    20:33:23 - INFO - Guard is now watching at '/var/www/StatsDClientBundle'



This will test the entire library for you, any changes will test only the files that have changed

If you install `libnotify` with

`gem install libnotify`

you will have an Icon next your top panel.


### 4 Trick Guard on large directories

If you have big project you need to increase the amount of watches of libnotify

see this page for more info: https://github.com/guard/listen/wiki/Increasing-the-amount-of-inotify-watchers



## `Inspect`  Guard!

Another plugin that I suggest is Guard for code inspection:

is easy to install


    sudo pear install PHP_CodeSniffer
    sudo gem install guard-phpcs


and then add to your Guardfile


    guard 'phpunit', :tests_path => 'Tests', :cli => '--colors' do
     # Watch tests files
     watch(%r{^.+Test\.php$})
     # Watch library files and run their tests
     watch(%r{^/(.+)/(.+)/(.+)\.php}) { |m| "Test/#{m[1]}/#{m[2]}/#{m[3]}Test.php" }
    end
    guard 'phpcs', :standard => 'PSR1' do
        watch(%r{.*\.php$})
    end


This will detect all the camel case problems, and you will have to follow the standard PSR1 :D


##  Guard + `Symfony2` = guard-phpunit-sf2

Unfortunately `guard-phpunit` does not work with symfony2 framework,

so I forked that repo and I made [guard-phpunit-sf2](​​https://github.com/liuggio/guard-phpunit-sf2)

**Hurray!!**

guard-phpunit-sf2 has not been pushed on RubyGems. If you want to try it, clone the repository, build the gem and install it.


    git clone git@github.com:liuggio/guard-phpunit-sf2.git
    cd guard-phpunit-sf2
    gem build guard-phpunit-sf2.gemspec
    sudo gem instal guard-phpunit-sf2-0.1.4.gem


then create the Guardfile into the root of your Symfony2 project

    guard 'phpunit', :cli => '--colors -c app/' do
         # Watch tests files
         watch(%r{^src\/.+Test\.php$})

         # Watch src file and run its test,
         # Test string: src/Tvision/Bundle/CartBundle/Repository/CartRepository.php
         watch(%r{^src\/(.+)\/(.+)Bundle\/(.+)\.php$}) { |m| "src/#{m[1]}/#{m[2]}Bundle/Tests/#{m[3]}Test.php" } # Watch all files in your bundles and run the respective tests on change
    end

then lunch `guard` and start testing.


PS: maybe you should uninstall guard-phpunit in order to work with symfony2 :|

`sudo gem uninstall guard-phpunit`


Happy test to everybody.



