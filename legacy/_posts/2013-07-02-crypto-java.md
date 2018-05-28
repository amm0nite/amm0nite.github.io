---
layout: post
title: "Chiffrement asymétrique avec Java"
published: true
categories: programming
---

Je suis en train de développer un logiciel de P2P en Java pour le fun. Une exigence de mon projet est la confidentialité de la communication et l'authentification du correspondant. J'ai choisi la cryptographie asymétrique (clé privée - clé publique) comme solution, mais comment l’implémenter simplement en Java?

Bonne nouvelle tout est inclus dans l'API Java! Voilà un exemple simple :

{% gist 5379572 %}

Avec RSA on ne chiffre que des données de petite taille car l'opération est longue (et plus complexe sur des données de grande taille). La bonne pratique est d'utiliser la cryptographie asymétrique pour échanger une clé secrète de cryptographie symétrique qui sera utilisée pour chiffrer les données à échanger.

{% gist 5379594 %}

On souhaite envoyer des messages d'un utilisateur à un autre, tout en étant sûr que ceux qui interceptent le message ne peuvent pas le lire (confidentialité), et que le message ne peut pas être intercepté et renvoyé par un autre (authentification de l'auteur par le récepteur). On part du principe que tous les utilisateurs possèdent une paire de clés et connaissent seulement la clé publique de chaque autre utilisateur.

Dans l’exemple précédent, la confidentialité est présente mais pas l'authentification. On va programmer l'auteur du message pour qu'il chiffre une signature dans le message avec sa clé privée, ensuite le destinataire vérifie cette information en la déchiffrant avec la clé publique de l'auteur. Si l'information peut être déchiffrée et qu'elle est correcte, cela veut dire que l'auteur possède bien la bonne identité (clé privée).

Attention, dans ce cas là, un utilisateur qui intercepte le message peut déchiffrer l'information (car tout le monde a les clés publiques), et la chiffrer avec sa clé privée. Ainsi l'intercepteur peut réutiliser le message à son compte. C'est pourquoi on va protéger la signature chiffrée avec la cryptographie symétrique de la même manière que les données.

La signature est un hash des données (SHA1 par exemple) qui permet de vérifier l'intégrité du message (il n'a pas été modifié sur le trajet).

{% gist 5379620 %}

Maintenant on a de quoi jouer avec le chiffrement asymétrique!