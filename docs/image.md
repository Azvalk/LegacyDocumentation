# Image

Le [GraphicComponent](graphicComponent.md) le plus frequent.  
Capable d'etre visible sous la forme d'un `sf::RectangleShape` avec texture.

## Fonctionnement:

En lui attribuant un nom de texture, l'Image va automatiquement trouver la texture dans les ressources, et l'assigner à son rectangleShape.  
A partir du moment où le sprite est actif, il est altéré par un `sf::IntRect`.  
Ce meme rect qui reçoit de nouvelle valeur en provenance de l'[Animator](animator.md).

##Exemple:

```c
//Creation et ajout du Renderer
auto playerRenderer = m_player->AddComponent<Renderer>();
//Creation de l'Image
auto playerImage = GraphicComponent::CreateGraphicComponent<Image>();
//Setup de l'Image
playerImage->SetTextureName("shroom");
//Ajout de l'Image a l'objet, en tant que GraphicComponent
m_player->AddComponent<GraphicComponent>(playerImage);
```

Il est tres important que l'Image soit ajoutée à l'objet en tant que GraphicComponent,  
et non en qu'en qu'Image. Sinon il ne sera pas trouvé par le Renderer.  
C'est pour cela que sa creation et son ajout sont 2 opérations distinctes,  
dans l'exemple visible en [Scene](scene.md).  
L'image est créée par la fonction `CreateGraphicComponent<Image>`,  
afin de l'avoir en type Image et donc utiliser sa fonction `SetTextureName()`, mais ensuite, elle est ajoutée en tant que GraphicComponent, par `AddComponent<GraphicComponent>(_image)`