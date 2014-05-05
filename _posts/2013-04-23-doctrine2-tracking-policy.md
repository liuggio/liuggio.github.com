---
layout: post
title: "Doctrine2 tracking policy"
description: "Tracking policy"
category: post
tags: [doctrine2, PHP]
published: true
edit-link: https://github.com/liuggio/liuggio.github.com/edit/master/_posts/2013-04-23-doctrine2-tracking-policy.md
---
{% include JB/setup %}

# Which tracking policy are you using?

With doctrine you can change the tracking policy for each entity, the tracking policy is the way in which
 Doctrine understands if a entity property must be 'saved' in the database.

Doctrine definition:
"Change tracking is the process of determining what has changed in managed entities since the last time they were synchronized with the database."

There are 3 tracking policies: Implicit, Explicit, Nofity.

## Implicit tracking (the default one)

The implicit is the default one and the slowest, you don't need to explicit call 'persist' eg.

` // $entity is not a new object, and has $entity->Title = 'liuggio loves tvision.github.io'`

` $entity->setTitle('liuggio loves symfony2');`

` $entityManager->flush();`

` echo $entity->getTitle();   // output 'liuggio loves symfony2'`


Unit of work now has **$entity.title=liuggio loves symfony2**

Database has flushed without persist **$entity.title=liuggio loves symfony2**

**Pro**:    `$entityManager->persist($entity)` is not necessary

**Cons**: Slowness,  Doctrine has to check property-by-property for all the object in the unit of work

## The explicit tracking (the suggested one)
You have to explicit flag which object you want to persist, than Doctrine will check property-by-property in order to find with property has changed.

**Pro**: Faster than implicit, must if you have a lot of entities.

**Cons**: the object in the unit of work could be different from that one in the db, not easy to debug.

The example

` // $entity is not a new object, and has $entity->Title = 'liuggio loves tvision.github.io'`

` $entity->setTitle('liuggio loves symfony2');`

` $entityManager->flush();`

` echo $entity->getTitle();   // output 'liuggio loves symfony2'`

Unit of work  now has **$entity.title=liuggio loves symfony2**

Database has different state for that entity **liuggio loves tvision.github.io**


## Notify tracking

It's not really a tracking policy, if you want to develop your own tracking policy, you have the tool for doing it.

Don't understimate this policy useful if you a domain logic.

## Link to doctrine doc

[doctrine/change-tracking-policies](http://docs.doctrine-project.org/en/latest/reference/change-tracking-policies.html)


