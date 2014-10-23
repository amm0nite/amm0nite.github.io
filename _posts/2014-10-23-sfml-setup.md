---
layout: post
title: "Démarrer avec SFML sur Visual Studio 2013"
published: true
categories: gamedev
---

Pour pouvoir développer avec SFML sur Visual Studio 2013, il faut compiler les sources et bien configurer l'IDE. 
Je fait un article histoire d'avoir une référence pour refaire cette manipulation rapidement.

Cet article est en fait une synthèse de deux tutoriels oficiel de SFML.

* [Compiler avec CMake](http://sfml-dev.org/tutorials/2.0/compile-with-cmake-fr.php "Compiler avec CMake")
* [SFML et Visual studio](http://sfml-dev.org/tutorials/2.1/start-vc-fr.php "SFML et Visual studio")

## Compiler SFML

[Télécharger](http://sfml-dev.org/download-fr.php) le code source de SFML

Créer les dossiers suivants:

    C:\SFML\source
    C:\SFML\build
    C:\SFML\install

Extraire le contenu de l'archive dans le dossier `SFML/source`

![Contenu du dossier "source"]({{ site.url }}/assets/source.png)

[Télécharger](http://www.cmake.org/download/) et installer CMake (avec Win32 Installer), sélectionner "Add CMake to the system PATH".

![Ajouter au system PATH]({{ site.url }}/assets/cmake.png)

## Configurer Visual Studio

REDACTION EN COURS 