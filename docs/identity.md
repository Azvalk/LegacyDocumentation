# Identity

Le [GameComponent](gameComponent.md) Identity est present de base dans tout [GameObject](gameObject.md).
Il est le porteur du nom, et des tags du GameObject.

## Fonctionnement:
Lors de la création d'un GameObject, vous pouvez lui donner un nom, et un tag principal.  
Des valeurs par defaut sont en place dans la fonction de création, ce qui rend cela optionnel.  
Mais il est recommandé de le faire, car cela est utile aux fonctions de recherche d'objet dans la hierarchie.  

Identity étant obligatoire, la class GameObject possède les fonctions pour agir dessus,  
sans avoir besoin de `GetComponent<Identity>`

Voir la page [GameObject](gameObject.md) pour la liste des fonctions de recherche d'objets.  

* Création d'un objet nommé "world", avec le tag principal "WORLD"
```C++
m_world = GameObject::CreateGameObject("world", "WORLD_1");
```
---
* Ajout d'un tag "level_1"
```C++
m_world->AddTag("level_1");
```
---
* Questionnement de la presence du tag "level_1" sur l'objet
```C++
if(m_world->HasTag("level_1"))
{
    //...
}
```
* Dans l'update d'un GameComponent, puisque l'objet servant de monde est disponible, recherche d'un ennemi particulier, nommé "enemy_Monster_1", dans le world.  
(Mauvaise pratique de rechercher un objet en update, juste pour l'exemple)
```C++
void Update(std::shared_ptr<GameObject> _world)
{
    auto enemy = GameObject::GetChildByName("enemy_Monster_1", _world);
    if(enemy != nullptr)
    {
        auto enemyComponent = enemy->GetComponent<Enemy>();
        if(enemyComponent != nullptr)
        {
            //..
        }
    }
}
```