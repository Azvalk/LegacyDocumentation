# Description

## Qu'est-ce que le Legacy ?

Le Legacy est un "moteur", de type surcouche d'api.     
En l'occurence, une surcouche de la [Sfml](https://www.sfml-dev.org/index-fr.php).

Le Legacy, fait suite à un précédent moteur de meme type, le KME, ou Knighmare Engine,  
initialement conçu à l'occasion du développement du jeu [Knight'Mare](https://www.gameacademy.fr/projets/knightmare).  
Sa conception s'inspirait d'un modele, Unity.

Un systeme de Scene, l'utilisation évidente du design pattern Composite,  
ainsi qu'un Ressource Manager, sont les 3 élements fondamentaux du programme.

Le Legacy, reprend tout ce que le KME a réussi, et met également à l'oeuvre toutes les connaissances et compétenceces acquises en pratiquant Unity.  
Le développement du Legacy se veut plus propre, plus respectueux des principes S.O.L.I.D,  
et plus modulaire, de part son découpage en plusieur DLL.

## Les DLL
Le Legacy est en effet découpé en plusieurs modules, tous indépendants les uns des autres,  
mais tous, dépendant d'une DLL noyau, **LegacyLibraryCore**, et de la **Sfml**.

### LegacyCore

Le noyaux du moteur, est constitué des elements fondamentaux. A savoir:

1. [**SfmlWindow**](sfmlWindow.md):
    - Wrapper d'une Window de Sfml. Permet de la controler facilement.
2. [**Scene**](scene.md)
    - Class mère, et abstraite, du systeme de Scene.
3. [**GameLoop**](gameLoop.md)
    - Responsable du cycle de vie du jeu.
4. [**GameObject**](gameObject.md)
    - Un objet disposant de base, d'un Transform et d'un Identity.
    - Peut avoir d'autre GameObject en enfant
    - Peut disposer d'une variété de GameComponent
5. [**GameComponent**](gameComponent.md)
    - Class mere des composant d'objet.
    - Libre a vous de créer des class enfants pour créer les comportements que vous voulez.
6. [**Transform**](transform.md)
    - Composant responsable de la position, rotation et scale de l'objet
    - Wrapper du Transform de la Sfml
7. [**Identity**](identity.md)
    - Composant permettant a un GameObject de disposer d'un nom, d'un Tag principal, et de tout une liste de Tag.
8. [**Renderer**](renderer.md)
    - Composant responsable de l'affichage d'un GameObject
9. [**Graphic Component**](graphicComponent.md)
    - Composant visuel abstrait, rendu par le Renderer.
    - Des class enfant concretes existent.
10. [**Animator**](animator.md)
    - Composant à la base du systeme d'animation.
    - D'autres elements important constitue ce systeme.
11. [**TimeTool**]()
    - Class statique disposant des outil de temps de la Sfml
12. [**RessourceManager**]()
    - Composant responsable des ressources du jeu.
    - Fait parti d'un systeme complexe de chargement de ressource en multithread.

### LegacyInput

Dll contenant le systeme d'input.
Il est prévu pour gérer les input clavier/souris ainsi que manette.  
Mais sa conception est encore à ce jour sujet à reflexion

1. [**Input Source**]()
    - Class abstraite ayant le role de source d'input.
    - 2 class enfant existent : **SfmlInputSource**, **GameXPadInput**
    - La différence est dans le mapping des touches par defaut.
2. [**gamepadx**]()
    - La petite lib externe permettant d'avoir acces au input d'une manette type Xbox.  

### LegacyUI

Dll contenant le systeme d'UI.
Pour l'heure cette dll n'existe pas, le systeme est encore en 
développement dans un projet de test.
