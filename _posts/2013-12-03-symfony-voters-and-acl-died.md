---
layout: post
title: "Symfony Voters and ACL died"
description: ""
category: 
tags: []
published: false
---
{% include JB/setup %}

C'è una funzionalità nascosta e non ben pubblicizzata dalla documentazione di Symfony2 i voters.

Due anni fa abbiamo iniziato a sviluppare il nostro bel e-commerce [Terravision](http://terravision.eu) in Symfony2,
al tempo Symfony2 si chiamava Symfony-reloaded e non era stable.

Al tempo per decidere chi potesse fare cosa abbiamo utilizzato le ACL, beh dopo un mese abbiamo tolto le ACL.

Abbiamo avuto non solo molte eccezioni, avere un sistema che potesse decidere nel micro sul singolo oggetto chi può e chi non può
è molto utile, ma ha forti svantaggi sia di performance che di rotture di cazzo.

Una alternativa c'è i Voters.

I voter sono classi votanti che proprio come succede nel parlamento, dicono la loro se un utente puo' effettuare.

<blackquote>
A voter is a dedicated class that checks if the user has the rights to be connected to the application
<small>ref. sf2-cookbook</small>
</blockquote>


La prima volta che ho letto la documentazione conoscendo le ACL ho pensato che i voters fossero qualcosa di basso livello solo per smanettoni.


A custom voter must implement [VoterInterface](http://api.symfony.com/2.4/Symfony/Component/Security/Core/Authorization/Voter/VoterInterface.html),
which requires the following three methods:

    interface VoterInterface
    {
        public function supportsAttribute($attribute);
        public function supportsClass($class);
        public function vote(TokenInterface $token, $object, array $attributes);
    }


**todo** Cosa prende in input supportsAttribute

**todo** Cosa prende in input supportsClass

Tutto molto facile se ci serve di bloccare qualcosa che già sappiamo esempio l'ip, o un particolare utente, ma le cose si complicano se dobbiamo gestire una sorta di access list in base ad una risorsa.

La bellezza del voter è che aiuta molto i progetti che hanno bounded context definiti, come ad esempio un progetto che segue la filosofia del SOA, potrebbe sviluppare un voter per ogni servizio. E' un ottimo modo per decentralizzare.

Symfony\Component\Security\Core\Authorization\Voter\RoleVoter







#### References

[sf2-cookbook Symfony/Cookbook/Security/Voters](http://symfony.com/doc/current/cookbook/security/voters.html)