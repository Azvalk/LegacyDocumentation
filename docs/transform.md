# Transform

Le composant Transform...
A connu de nombreuses versions au cour du développement.
Finalement, il semble stabilisé dans une forme suffisament générique pour repondre à tout les besoins.
Aussi bien pour les GameObject de "gameplay", que ceux d'UI.

### Fonctionnement:

Un Transform dispose de:
- un sf::FloatRect box.
- un sf::Vector2 position.
- un sf::Vector2 scale
- un sf::Vector2 origin
- un float rotation.

La box est utilisée par le component Image pour déterminé la taille du rect affiché.
L'origine est également prise en compte.
Pour un jeu 2D profil, il est donc recommandé de la placer en bas au centre des personnages, à ses pieds.

La box est également un element qui sera important à prendre en compte dans un contexte d'UI.

Les quelques fonctions dont ils disposent sont assez intuitives, mais rien ne vaut la pratique pour comprendre toute les fourberies qu'ils pourraient vous réserver.