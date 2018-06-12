---
layout: post
title: "Démarrer avec SFML sur Visual Studio 2013"
published: true
categories: gamedev
---

Pour pouvoir développer avec SFML sur Visual Studio 2013, il faut compiler les sources et bien configurer l'IDE. 
Je fait un article histoire d'avoir une référence pour refaire cette manipulation rapidement.

Cet article est en fait une synthèse sans explications de ces deux tutoriels officiels de SFML.

* [Compiler avec CMake](http://sfml-dev.org/tutorials/2.0/compile-with-cmake-fr.php "Compiler avec CMake")
* [SFML et Visual studio](http://sfml-dev.org/tutorials/2.1/start-vc-fr.php "SFML et Visual studio")

## Compiler SFML

[Téléchargez](http://sfml-dev.org/download-fr.php) le code source de SFML.

Créez les dossiers suivants:

    C:\SFML\source
    C:\SFML\build
    C:\SFML\install

Extrayez le contenu de l'archive dans le dossier `SFML/source`. Vous devez obtenir ceci dans le dossier `source` :

![Contenu du dossier "source"]({{ site.url }}/assets/images/source.png)

[Téléchargez](http://www.cmake.org/download/) et installez CMake (avec Win32 Installer), sélectionnez "Add CMake to the system PATH".

![Ajouter au system PATH]({{ site.url }}/assets/images/cmake.png)

Lancez la console développeur pour VS2013. Le raccourci pour accéder à la console se situe dans ce dossier `C:\Program Files (x86)\Microsoft Visual Studio 12.0\Common7\Tools\Shortcuts` (accessbile depuis le "menu démarrer" dans le dossier "Visual Studio Tools").

Lancez l'interface de CMake en tapant `cmake-gui`.

Remplissez les deux premiers champ avec les bon dossiers  `source` et `build`.

![Dossier source et build]({{ site.url }}/assets/images/paths.png)

Cliquez sur le bouton "Configure", choisissez "NMake Makefiles" et vérifiez que "Use default native compilers" est activé.

![Configure: première étape]({{ site.url }}/assets/images/configure1.png)

Après avoir validé, les variables de configuration apparaissent sur fond rouge. Modifiez les variables comme dans la capture ci dessous.

![Configure: deuxième étape]({{ site.url }}/assets/images/configure2.png)

Cliquez une nouvelle fois sur "Configure" et les options devraient passer de rouge à blanc. Cliquez ensuite sur le bouton "Generate".

Lancez de nouveau la console développeur pour VS2013. Se déplacer dans le dossier `build` en tapant `cd C:\SFML\build`.

Compiler SFML en tapant `nmake` et ensuite installez les fichiers avec `nmake install`.

Les fichiers d'entête, les .lib et les .dll dont VS a besoin se trouvent maintenant dans le dossier `install`.

## Configurer Visual Studio

Lancez Visual Studio 2013 et créez votre projet avec le template de projet vide C++.

Ouvrez les propriétés du projet (clic droit sur le nom du projet dans l'explorateur de solution) pour configurer l'emplacement des fichiers nécéssaires.

![VS2013 propriétés du projet]({{ site.url }}/assets/images/vs1.png)

Dans `Propriétés de configuration > C/C++ > Général > Autres répertoires Include`, ajoutez le repertoire `C:\SFML\install\include`.

![include]({{ site.url }}/assets/images/include.png)

Dans `Propriétés de configuration > Editeur de liens > Général > Répertoires de bibliothèques supplémentaires`, ajoutez le repertoire `C:\SFML\install\lib`.

![lib]({{ site.url }}/assets/images/lib.png)

Dans `Propriétés de configuration > Editeur de liens > entrée > Dépendances supplémentaires`, ajoutez :

    sfml-audio.lib
    sfml-graphics.lib
    sfml-main.lib
    sfml-network.lib
    sfml-system.lib
    sfml-window.lib
    
![dll]({{ site.url }}/assets/images/dll.png)

Pour tester, créez un fichier `main.cpp` avec le code suivant puis compilez le projet (en mode "Release").

{% highlight c++ %}
#include <SFML/Graphics.hpp>

int main()
{
    sf::RenderWindow window(sf::VideoMode(200, 200), "SFML works!");
    sf::CircleShape shape(100.f);
    shape.setFillColor(sf::Color::Green);

    while (window.isOpen())
    {
        sf::Event event;
        while (window.pollEvent(event))
        {
            if (event.type == sf::Event::Closed)
                window.close();
        }

        window.clear();
        window.draw(shape);
        window.display();
    }

    return 0;
}
{% endhighlight %}

Un message d'erreur va apparaître car il faut au préalable copier les fichiers DLL de SFML dans le même dossier que l'éxécutable du projet.

Copiez tout les fichiers DLL de `C:\SFML\install\bin` dans le dossier du projet qui contient `VotreProjet.exe`.

Et maintenant tout fonctionne, lisez ce [livre](https://www.packtpub.com/game-development/sfml-game-development) et programmez un jeu!