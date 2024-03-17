# Scene

Une Scene, est un objet ayant pour role de servir de "situation globale".  
On peut le voir comme un niveau, ou comme un état, mais s'il porte le nom de Scene, c'est parce que le plus intuitif est de le voir comme une Scene de theatre, en plus métaphorique.  
Les objets qui agiront dedans en seront les acteurs.

Sur Legacy, certain choix de design ont été fait concernant l'objet Scene.  
Il aurait pu en etre autrement, mais il fallait bien se décider pour quelque chose.

Ainsi, une Scene contient par defaut, un [GameObject](gameObject.md) nommé **Root**.
Il est l'objet racine, la source de la hierarchie d'objets qui seront mis en scene.  
Cet objet Root, au sommet de la hierarchie, est accessible par tout ses enfants,  
grace à une fonction qui remonte jusqu'à lui: <font size = 4>**`GetRoot()`**.</font>  
Cette facilité d'access octroie le droit à cet objet, d'avoir un role un peu plus étendu que simplement le sommet de la hierarchie.  
Il est l'objet qui porte le [RessourceManager](), et plus tard le [NetworkManager]().

---
## Créer une Scene
1. Créer une class héritée de **lg::Scene**
2. Implémenter les fonctions virtuelles abstraites.
3. Implémenter un override de la fonction Init:  **virtual void Init() override**;
4. Votre fonction Init() doit premierement faire appel à **lg::Scene::Init()**, sans quoi, la Scene n'aura pas d'objet Root.
5. C'est dans cette fonction Init, que vous pouvez créer les GameObject initiaux de la scene, et organiser la hierarchie.
6. Ne pas oublier de placer le tout, dans l'objet Root.

---
## Exemple d'Init complet:
```C++
void GameScene::Init()
{
	Scene::Init();
    auto root = GetSceneRoot();

	/*WORLD*/
    m_world = GameObject::CreateGameObject("world", "WORLD");
	auto wr = m_world->AddComponent<Renderer>();
	auto worldBackground = GraphicComponent::CreateGraphicComponent<Image>();
	worldBackground->SetTextureName("background");
	m_world->AddComponent<GraphicComponent>(worldBackground);

	/*PLAYER*/
	m_player = GameObject::CreateGameObject("player", "PLAYER");
	//Component principal:
	auto playerComponent = m_player->AddComponent<Player>();
	//Control:
	auto controller = m_player->AddComponent<PlayerController>();
	controller->SetPlayer(playerComponent);
	//Rendu:
	auto playerRenderer = m_player->AddComponent<Renderer>();
	auto playerImage = GraphicComponent::CreateGraphicComponent<Image>();
	playerImage->SetTextureName("shroom");
	m_player->AddComponent<GraphicComponent>(playerImage);
	//Animation:
	auto playerAnimator = m_player->AddComponent<Animator>();
	auto playerAnimationController = new AnimationController_Player(playerAnimator);
	playerAnimator->SetAnimationController(playerAnimationController);
	//Positionnement
	m_player->GetTransform()->setOrigin(24.f, 24.f);
	m_player->SetPos(SCREEN_W / 2.f, SCREEN_H / 2.f);
	
	//Hierarchie
    m_world->AddGameObject(m_player);
    root->AddGameObject(m_world);

	SetIsLoading(true);
	LoadScene();
}
```

Dans cet exemple, on voit:

* Init de Scene
* Recupération du root (car il est privé à la class mere)
* Création d'un GameObject "World". Cet objet fait partie des variables de cette class de Scene. ( variable membre reconnaissable à m_)
* Création et ajout, du [Renderer](renderer.md), pour l'objet world.
* Création, ajout et réglage du [GraphicComponent](graphicComponent.md) du world, en l'occurence une [Image](image.md)
* Création d'un GameObject Player.
* Création et ajout d'un GameComponent Player
* Création, ajout, du Renderer du Player.
* Création, ajout et réglage du GraphicComponent du player, aussi une Image. 
* Réglage sur le [Transform](transform.md) du Player.
* Ajout du player au world
* **Ajout du world au root.**
* La scene passe en mode loading actif. Important pour la gestion de la Scene par la GameLoop.
* Utilisation de la fonction **LoadScene**. Le Ressourcemanager contenu dans le Root va faire son oeuvre.

---
## Exemple de LoadScene:
```C++
void GameScene::LoadScene()
{
	LoadingProcess::loadTextures(GetRessourceManager(), "Textures");
	LoadingProcess::loadAllFontsOfFolder("..\\Ressources\\Fonts", "All", GetRessourceManager());
	LoadingProcess::loadAllSoundsOfFolder("..\\Ressources\\Sounds", "All", GetRessourceManager());
	SetReady(true);
}
```
Le [LoadingProcess]() est une class disposant uniquement de fonctions static.  
Elles fouillent les dossiers qu'on leur donne, et fournissent les chemins des fichiers trouvés,  
au RessourceManager qu'on leur passe en argument.
---
## Exemple d'Update:
```C++
void GameScene::Update(sf::RenderWindow& _renderWindow)
{
	if (IsLoading())
	{
		if (!GetRessourceManager()->getLoader()->isActive())
		{
			SetIsLoading(false);
		}
	}
	else
	{
		keyTimer += TimeTool::GetTimeDelta();
		if (GetSceneRoot() != nullptr)
			GetSceneRoot()->Update(m_world);
		//On peut s'arreter la

		//Un test de passage vers une autre scene
		if (sf::Keyboard::isKeyPressed(sf::Keyboard::A) && keyTimer > 0.2f) 
		{
			auto newScene = new SecondGameScene("SecondGameScene");
			newScene->SetUseRessourceManagerFromPreviousScene(true);
			SetNextScene(newScene);
			keyTimer = 0.f;
		}
	}
}
```
---
## Exemple de HandleEvent:
```C++
void GameScene::HandleEvent(const sf::Event& _event, sf::RenderWindow& _window)
{
	GetSceneRoot()->EventHandle(_event, _window);
}
```
---
## Exemple de Display:
```C++
void GameScene::Display(sf::RenderWindow& _renderWindow)
{
	if (GetSceneRoot() != nullptr)
		GetSceneRoot()->Draw(_renderWindow, sf::RenderStates());
}
```
---
Voyez qu'à aucun moment, les fonctions Update, HandleEvent, et Display, n'ont la moindre idée qu'un world et un player
font partie de la scene.  

- L'Update fait son role, faire tourner le "moteur update" de l'objet **Root**
- L'HandleEvent, de meme, transmet l'event au Root.
- Le Display de meme, demande au Root de se draw.