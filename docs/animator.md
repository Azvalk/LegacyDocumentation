# Animator

L'Animator, est le [GameComponent](gameComponent.md) dont va avoir besoin un GameObject,
pour animer son [GraphicComponent](graphicComponent.md), le plus souvent une [Image](image.md), et donc le rectangleShape détenu par cette Image.

### Fonctionnement:
L'Animator dispose d'un [AnimationController](animationController.md), une machine à état qui utilisera des [AnimationState](animationState.md).
Chaque AnimationState aura une [Animation](animation.md), et chaque Animation aura au minimum 1 **Sequence** de frames, mais pourrait en avoir plusieurs.  
La finalité, étant qu'une Animation fournit un `sf::IntRect`, qui sera communiqué à l'Image par l'Animator uniquement à la condition qu'il ait changé.

 <font size = 5>**`Update:`**</font>
```c
void Animator::Update(std::shared_ptr<GameObject> _world)
{
	if (auto image = m_linkedImg.lock())
	{
		if (m_animationController->Update()) //Renvoi true si la frame a changé
			image->SetFrameRectByRef(m_animationController->GetCurrentFrame());
	}
}
```

### Equiper un animator sur un GameObject:
```c
//Ajout de l'animator à l'objet, et conservation en variable le temps de le setup
auto playerAnimator = m_player->AddComponent<Animator>();
//Creation d'une instance d'animation controller, de la class héritée que l'on souhaite
auto playerAnimationController = new AnimationController_Player(playerAnimator);
//Confier le controller à l'animator.
playerAnimator->SetAnimationController(playerAnimationController);
```
