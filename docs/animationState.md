# Animation State

### Fonctionnement:
Un state dispose de plusieurs "phase", ou "evenement".  

- Enter : Le state commence, il s'assure de jouer son animation du début.
- End : Le state est fini, car son animation a atteint sa frame final, et ne boucle pas.
- Exit : Le state n'est plus actif, on le quitte, certainement pour passer sur un autre.

Lorsque l'[AnimationController](animationController.md), passe d'un state à l'autre, 
ces fonctions sont utilisées:
```c
	std::function<void()> OnEnter;
	std::function<void()> OnExit;
	std::function<void()> OnEnd;
```

Un state est capable de récuperer le sf::IntRect actuel, la frame d'animation, présentement utilisée par l'animation qu'il joue.
Lorsqu'une frame change dans l'animation, le state est prevenu par un bool.
Ce signal remonte à travers l'update de l'AnimationController, jusqu'à etre visible par l'[Animator](animator.md), qui decide alors de transmettre ce nouveau sf::IntRect, à l'[Image](image.md).

---
### .cpp:
```c
#include "pch.h"
#include "AnimationState.h"
#include <iostream>

namespace lg
{
	AnimationState::AnimationState(std::string _name, Animation* _animation)
		: m_name(_name), m_animation(_animation), m_isPlaying(false), m_defaultRect(sf::IntRect(0, 0, 0, 0))
	{}

	AnimationState::~AnimationState()
	{
		delete(m_animation);
	}

	std::string& AnimationState::GetName()
	{
		return m_name;
	}

	void AnimationState::SetName(std::string _name)
	{
		m_name = _name;
	}

	sf::IntRect& AnimationState::GetCurrentFrame()
	{
		if (m_animation != nullptr)
			return m_animation->GetCurrentFrame();

		return m_defaultRect;
	}

	void AnimationState::SetAnimation(Animation* _animation)
	{
		if (m_animation != nullptr)
			delete(m_animation);

		m_animation = _animation;
	}

	void AnimationState::Enter()
	{
		m_animation->Reset();
		SetPlaying(true);
	}

	void AnimationState::End()
	{
		SetPlaying(false);
	}

	void AnimationState::Exit()
	{}

	Animation* AnimationState::GetAnimation()
	{
		return m_animation;
	}

	std::shared_ptr<AnimationState> AnimationState::CreateState()
	{
		return std::shared_ptr<AnimationState>(new AnimationState());
	}

	bool AnimationState::Update()
    {
        //Est-ce que la frame a changé au cours de cette update ?
	    bool frameChanged = false; //Non par defaut.
	    if (m_animation != nullptr && m_isPlaying)
	    {
            //L'Animation va avoir le droit d'editer ce bool
	    	if (m_animation->Update(frameChanged))
		    {
			    SetPlaying(false);
			    if (OnEnd)
				    OnEnd();
		    }
	    }
	    return frameChanged; //Et il est retourné ici
    }

	void AnimationState::SetPlaying(bool _value)
	{
		m_isPlaying = _value;
	}

	bool& AnimationState::IsPlaying()
	{
		return m_isPlaying;
	}
}
```