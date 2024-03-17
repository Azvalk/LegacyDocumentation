# Game Component

Un Game Component est un morceau de [GameObject](gameObject.md).
Son code est dédié à remplir une fonction, un role précis pour constituer le comportement de l'objet.  
Il detient des variables et une logique qui lui son propre, et peut communiquer avec les autres Game Component de l'objet.
Vous pouvez inventer ce que vous voulez en GameComponent, il suffit d'en heriter, et d'implementer la fonction virtuelle pure: **`Update`**.  

## Fonctionnement:

La caractéristique principale d'un GameComponent, c'est qu'il s'update à chaque frame.
C'est le GameObjet qui se charge de l'appel d'Update.  
Le code de son update fait donc partie de la boucle d'update du programme.  

La caractéristique secondaire, et qu'il peut également capter les evenements de la sfml.  
Il faut pour cela implementer un override de la fonction HandleEvent.
Utile par exemple, pour capter les evenements de souris ou clavier.  

## Fonctionnalités:
---
* <u>Création:</u>
```C++
template <typename T, typename... Args,
typename = typename std::enable_if<std::is_base_of<GameComponent, T>::value>::type>
static std::shared_ptr<T> CreateGameComponent(Args&&... args)
{
	return std::shared_ptr<T>(new T(std::forward<Args>(args)...));
}
```
Cette fonction static et templaté, sert à créer une instance de n'importe quel GameComponent.  
Elle donne également l'occasion de faire passer des arguments à son constructeur.

---
* <u>Communication:</u>

```C++
std::weak_ptr<GameObject> GameComponent::GetParent()
{
	return m_parent;
}
```
Lors de son ajout à un GameObject, le component s'attache à l'objet.  
Un pointeur faible le relie alors à cet objet, on l'appele le **parent**.  
Le pointeur est faible, car le component n'a pas autorité sur le GameObject,  
si l'objet disparait, le component ne doit pas avoir de reaction, car de toute façon la destruction du GameObjet, provoque la destruction de tout ses components.  

Puisque le pointeur est faible, lorsque le component souhaite établir le contact avec son objet parent pour communiquer avec les autres component, une petite manoeuvre syntaxique est necessaire pour établir le contact avec l'objet parent.

```C++
if (auto parent = GetParent().lock())
{
	//Code...
}
```
Par ce petit block if, on utilise la fonctionnalité du pointeur weak, qui est de se convertir en pointeur shared a la condition que le "pointage" soit toujours valide.
Autrement dit, si le lock echoue, car l'objet pointé n'existe plus, un nullptr sera renvoyé, et le block if ne sera pas exécuté.
Mais s'il réussi, la variable "parent", sera un pointeur shared sur le GameObject parent.
A partir de la, le `GetComponent<T>()` peut etre utilisé:

```C++
if (auto parent = GetParent().lock())
{
	auto player = parent->GetComponent<Player>();
}
```
**Important**:

* Pour que cette manoeuvre de lock du parent fonctionne, `#include <Legacy/GameObject.h>`
doit être présent dans le .cpp du component.
* Pour que le component à contacter soit récupérable par le `GetComponent<T>()`, il faut aussi l'include.
* Le mot clef auto est souvent utilisé car les GameObjet et GameComponent, sont sous la forme de `std::shared_ptr<T>`.
Et c'est long à écrire.  

* <u>Lien avec le monde</u>
```C++
virtual void Update(std::shared_ptr<GameObject> _world);
```
L'update des GameComponent, dispose d'un pointeur vers un GameObject.  
Il est recommandé de faire en sorte, que cet objet soit le "monde".  
Autrement dit, dans votre scene, préparez un objet pour incarner le monde, le "terrain de jeu", mettez le en enfant du Root de la scene.  
Les gameObject participants, incarnés dans ce monde, seront ses enfants.  
Ainsi, grace à ce pointeur de _world disponible en update, une recherche de gameObject peut être effectuée:

```C++
const std::list<shared_ptr<GameObject>>& GetChildren() const;
static std::list<shared_ptr<GameObject>> GetChildrenByIdentity(shared_ptr<GameObject> _container, std::string identityName);
static shared_ptr<GameObject> GetChildByIdentityName(std::string _name, std::shared_ptr<GameObject> _container);
static std::list<shared_ptr<GameObject>> GetChildrenExcept(std::string _name, std::shared_ptr<GameObject> _container);
static std::list<shared_ptr<GameObject>> GetChildrenWithTag(const std::string _tag, std::shared_ptr<GameObject> _container);
```
Ces fonctions peuvent être utilisées pour rechercher un gameObject particulier, ou une liste de gameObject.

---
* <u>Activité:</u>
```C++
void GameComponent::SetActive(bool _bool)
{
	m_active = _bool;
	if (m_active)
		OnEnable();
	else
		OnDisable();
}
```
Un Game Component, peut être désactivé et réactivé à volonté par cette fonction.
Son update ne sera pas lancé par l'objet parent.  
Par contre, son traitement des event aura bien lieu.  
Donc si vous souhaitez le couper également, il faudra voir cela directement dans votre implémentation de la fonction HandleEvent.

* <u>Automatisme:</u>
```C++
virtual void OnAdd() {};
virtual void OnEnable() {};
virtual void OnDisable() {};
virtual void OnRemove() {}
```
Ces fonctions ne sont pas virtuelle pure, pous ne pas vous forcer à les implementer,  
mais elles sont disponibles à l'override.

* `OnAdd` est appelé lorsque le component est attaché a l'objet.
Utile pour initialiser des choses en récuperant des informations dans les autres component deja présent dans l'objet.
* `OnEnable` est appelé quand le component devient actif.
* `OnDisable` est appelé quand le component devient inactif.
* `OnRemove` est appelé quand le component est retiré de l'objet.
