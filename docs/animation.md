# Animation

### Fonctionnement:

Une animation, comporte au minimum 1 **Sequence**. (`std::vector<sf::IntRect>`).  
Mais elle peut en comporter une infinité.
Pour créer une sequence, utilisez la fonction:

#### `GenerateSequence:`
**Les argument à fournir sont:**

- _startX: la coordonnée pixel de depart horizontalement
- _startY: la coordonnée pixel de depart verticalement
- _frameWidth et _frameHeight: largeur et hauteur pixel d'une frame
- _number: le nombre de frame constituant la sequence. 

```c
std::vector<sf::IntRect> Animation::GenerateSequence(int _startX, int _startY, int _frameWidth, int _frameHeight, int _number, bool _horizontal)
{
	int x = _startX;
	int y = _startY;
	std::vector<sf::IntRect> sequence;
	for (size_t i = 0; i < _number; i++)
	{
		sf::IntRect frame = { (int)(_horizontal ? (x + (i * _frameWidth)) : x), (int)(!_horizontal ? y + (i * _frameHeight) : y)  , _frameWidth, _frameHeight };
		sequence.push_back(frame);
	}
	return sequence;
}
```
---
Un exemple est disponible dans [AnimationController](animationController.md)  
Une fois la séquence générée, il faut l'assigner à un index dans l'animation,  
par la fonction `SetSequence(_index, _sequence)`.  
Une animation range ses sequences dans un `std::unordered_map<int, Sequence>`.  
Le int en clef, peut tres bien selon votre convenance, être lié à un enum de votre choix.  
Un changement de séquence est le genre de chose qui s'ordonne depuis une StateMachine de comportement,  
un exemple sera fourni.

Avoir 2 séquences dans une meme animation, permet d'en avoir une pour la droite, et l'autre pour la gauche.  
Dans le contexte d'un jeu 2D de profil.

Pouvoir disposer d'un grand nombre de sequences, rend le systeme compatible avec une autre vue, par exemple en Top-Down, où les animations pourraient exister en 8 directions.

Pour ordonner à une animation de changer de séquence,  
utilisez la fonction `SetSequenceKey(_key)`

---
### .cpp:
```c
#include "pch.h"
#include "Animation.h"
#include "TimeTool.h"

namespace lg
{
	Animation::Animation(std::string _animName)
		: m_name(_animName), m_currentFrame(0), m_frameNumber(0), 
		m_frequency(0.15f), m_looping(false), m_timer(0.f),
		m_defaultRect(sf::IntRect(0, 0, 0, 0)), m_currentSequenceKey(0)
	{}

	void Animation::SetupAnimation(std::vector<sf::IntRect> _frames)
	{
		m_animationFrames = _frames;
	}

	sf::IntRect& Animation::GetCurrentFrame()
	{
		if (m_currentFrame < m_animationFrames.size())
			return m_animationFrames[m_currentFrame];

		return m_defaultRect;
	}

	void Animation::Reset()
	{
		m_currentFrame = 0;
		m_timer = 0.f;
	}

	int& Animation::GetCurrentFrameIndex()
	{
		return m_currentFrame;
	}

	bool Animation::Update(bool& _frameChanged)
	{
		if (m_currentFrame != m_frameNumber)
		{
			m_timer += TimeTool::GetTimeDelta();
			if (m_timer >= m_frequency)
			{
				m_timer = 0.f;
				m_currentFrame++;
				_frameChanged = true;

				TriggerFrameEvent();

				if (m_currentFrame == m_frameNumber)
				{
					m_currentFrame = m_frameNumber - 1;
					if (m_looping)
					{
						m_currentFrame = 0;
						return false;
					}
					return true;
				}
				return false;
			}
			return false;
		}
		return true;
	}

	void Animation::SetAnimationFrequency(float _frequency)
	{
		m_frequency = _frequency;
	}

	void Animation::CreateSequence(int _startX, int _startY, int _frameWidth, int _frameHeight, int _number, bool _horizontal)
	{
		int x = _startX;
		int y = _startY;
		for (size_t i = 0; i < _number; i++)
		{
			sf::IntRect frame = { (int)(_horizontal ? (x + (i * _frameWidth)) : x), (int)(!_horizontal ? y + (i * _frameHeight) : y)  , _frameWidth, _frameHeight };
			m_animationFrames.push_back(frame);
		}
		m_frameNumber = _number;
	}

	void Animation::SetFrameEvent(int _frameIndex, std::function<void()> _function)
	{
		m_frameEvents[_frameIndex] = _function;
	}

	void Animation::SetLooping(bool _value)
	{
		m_looping = _value;
	}

	void Animation::SetCurrentFrame(int _value, bool _resetFrequency, bool _triggerFrameEvent)
	{
		if (_value == m_frameNumber)
			return;

		m_currentFrame = _value;
		if (_resetFrequency)
			m_timer = 0.f;

		if (_triggerFrameEvent)
			TriggerFrameEvent();
	}

	void Animation::TriggerFrameEvent()
	{
		if (m_frameEvents.contains(m_currentFrame))
		{
			m_frameEvents[m_currentFrame]();
		}
	}

	void Animation::SetSequence(int _key, Sequence _sequence)
	{
		m_sequences[_key] = _sequence;
	}

	void Animation::SetSequenceKey(int _key)
	{
		m_currentSequenceKey = _key;
		m_animationFrames = GetSequence(m_currentSequenceKey);
		m_timer = 0.f;
		m_frameNumber = m_animationFrames.size();
	}

	std::vector<sf::IntRect>& Animation::GetSequence(int _key)
	{
		return m_sequences[_key];
	}

	//GENERATE_SEQUENCE(...)
    //VISIBLE EN EXEMPLE 

	void Animation::ActivateFirstSequence()
	{
		m_animationFrames = GetSequence(0);
		m_frameNumber = m_animationFrames.size();
		m_timer = 0.f;
	}
}
```