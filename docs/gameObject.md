# GameObject

La star du moteur, le GameObject est une entité constituée de [GameComponent](gameComponent.md).  
Chaque GameComponent, peut se résumer à un script dédié à un role bien précis.
C'est par le respect de la compartimentation des roles que l'on obtient un programme tout terrain.

Un GameObject peut disposer d'un grand nombre de GameComponent,  
mais jamais plusieurs du meme type.  
Sans GameComponent, un GameObject n'est qu'une coquille vide.

## Création:
Pour créer un nouveau GameObject, on passe par une fonction static de la class GameObject.
`GameObject::CreateGameObject("nom", "TAG_PRINCIPAL")`;

## Fonctionnement:
- Un GameObject est équipé de base, de 2 GameComponent obligatoire:
    - [Transform](transform.md)
    - [Identity](identity.md)


#### Identité:
- L'identité du GameObject est détenue par son component Identity
- Elle se compose d'un nom, d'un tag principal, et d'un ensemble de tag

#### Positionnement et Echelle: 
- La position et l'échelle de taille, sont gérées par le Transform.
- Le transform étant obligatoire pour un gameObject, des fonctions relai existent au niveau de GameObject:
    - `GetPosition()`
    - `SetPosition(sf::vector2f _pos)`
    - `GetScale()`
    - `SetScale(sf::vector2f _scale)`


### Relation au GameComponent:
`AddComponent<T>` et `GetComponent<T>` sont 2 des fonctions les plus importantes pour un GameObject.

- AddComponent: Par cette fonction, on ajoute un component a l'objet.  
2 versions existent:
    - AddComponent<T>();
        - Ici, aucun component existant n'est fourni, la création aura lieu dans la fonction d'ajout.
        - Recommandé pour les components qui ont un constructeur sans argument.
    - AddComponent(_component);
        - Ici, un component a été créé avant, et passe en argument pour simplement être ajouté à l'objet.
    
    Dans les 2 versions, la fonction renvoi le component en question.


**Exemple:**
```C++
//Creation d'un objet basique, vide..on sait juste que son nom sera "player", et que son tag principal sera "PLAYER"
auto playerObject = GameObject::CreateGameObject("player", "PLAYER");
//Creation et ajout du GameComponent Player
auto playerComponent = playerObject->AddComponent<Player>();
//Ce n'est qu'une fois le component player en place dans cet objet, que cet objet peut etre considéré comme le player,
//Avant ce n'etait qu'une coquille.
```

### Activité et Destruction:

```C++
void setEnable(bool _bool) { m_enabled = _bool; }
void setNeutralized(bool _bool) { m_neutralized = _bool; }
void setNeedDelete(bool _bool) { m_needDelete = _bool; }
```

* Actif: Update active, visible s'il a un rendu.
* Inactif: Update inactive, ne fait pas son rendu, reçoit toujours les évènements.
* Neutralisé: Update inactive, toujours visible s'il a un rendu, reçoit toujours les évènements.
* En attente d'un Delete: sera supprimé de la hierarchie à la prochaine boucle d'update.

Le systeme étant à base de shared_ptr, lorsque l'objet n'est plus réferencé par la hierarchie,  
plus rien ne pointe sur lui, donc sa destruction est déclenchée.

### Recherche d'objet:

* <u>Static:</u>
- `GetChildByName(_name, _containerObject)`
- `GetChildrenByName(_name, _containerObject)`
- `GetChildByMainTag(_mainTag, _containerObject)`
- `GetChildrenExcept(_name, _containerObject)`
- `GetChildrenWithMainTag(_mainTag, _containerObject)`
- `GetChildrenWithTag(_tag, _containerObject)`
- `GetPlayer(_containerObject)`  

En fournissant un nom, un tag principal, ou un tag, et un GameObject dans lequel chercher,  
on peut generalement trouver ce que l'on cherche.  
Le GetPlayer va chercher un objet avec le tag principal "PLAYER".  
**Important**: ces recherches n'ont lieu que dans les enfants immédiat du container,  
et n'iront pas chercher dans les enfants des enfants.
---
* <u>Non Static</u>
- `GetRoot()`

GetRoot permet à tout moment de retrouver le Root de la [Scene](scene.md), utile pour acceder au component généraux, tel que le [RessourceManager](), ou le [NetworkManager]().