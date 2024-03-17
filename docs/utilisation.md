# Utilisation

Une fois le Legacy installé dans votre projet, la 1ere étape sera de faire votre main().  
En voici un exemple fonctionnel :

```C++
#include <Legacy/SfmlWindow.h>
#include <Legacy/Scene.h>
#include <Legacy/GameLoop.h>
#include "GameScene.h"

int main()
{
	auto window = lg::SfmlWindow::CreateDefaultWindow("Game");
	auto scene = lg::Scene::CreateScene<GameScene>("GameTest");
	scene->Init();
	auto gameLoop = lg::GameLoop(&window);
	gameLoop.Run(scene);
	return 0;
}
```

1. Création d'une window, par la fonction simplifiée.
Il existe d'autres fonctions pour créer une window en donnant des parametres.
Une fois la window créée, vous pouvez toujours utiliser certaines fonctions pour l'altérer.
2. Création d'une Scene.
Cette étape nécessite que vous créeriez une class héritée de Scene.
Dans l'exemple il s'agit de GameScene.
3. Initialisation de la Scene. 
Comme vous pouvez le voir en détail sur la page [Scene](scene.md), une scene a besoin d'être initialisée avant d'être utilisée.
4. Création de la [GameLoop](gameLoop.md).
Comme expliqué à sa page, la GameLoop conserve l'adresse de la window,
afin de la rendre accessible à ses phases.
La fonction run prend le pointeur de la scene.

Avec tout cela, si vous avez correctement préparé votre Scene, vous devriez avoir un résultat.