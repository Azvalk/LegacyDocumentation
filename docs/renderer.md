# Renderer

Le Renderer, est le [GameComponent](gameComponent.md) nécessaire à l'affichage du visuel d'un [GameObject](gameObject.md).  
Lors de sa phase d'affichage, un GameObject fait appel à son Renderer, qui fait appel à la fonctionnalité de draw du [GraphicComponent](graphicComponent.md) auquel il s'est lié dans le GameObject.  
GraphicComponent est un GameComponent abstrait, il en existe plusieurs type concrets, tels que les [Image](image.md) et les [Text](text.md).