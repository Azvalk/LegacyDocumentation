# Animation Controller

Un AnimationController, est une class construite comme une state machine.  
C'est une class abstraite dont vous allez devoir créer vos propres classes héritées.  
Par exemple un **AnimationController_Player**.  
Vous y établirez alors tout les [AnimationState](animationState.md) dont vous aurez besoin, et pourrez lier les état aux fonctions voulues.

## **Exemple d'AnimationController:**
### .h :
```c
#pragma once
#include <Legacy/AnimationController.h>

using namespace lg;
class AnimationController_Player : public AnimationController
{
public:
	AnimationController_Player(std::shared_ptr<Animator> _animator);
	
	//Fonction héritée de AnimationController, obligatoire à definir car virtuelle pure à la base.
	virtual void InitAnimationController() override;

	//Fonctions personnelles de ce controller pour organiser sa procédure
	void InitAnimationStates();
	void InitAndAssigneAnimations();

	//Les états
	std::shared_ptr<AnimationState> idleState;
	std::shared_ptr<AnimationState> walkState;

	//Les fonctions liées aux phases des etats
	void OnEnter_Idle();
	void OnUpdate_Idle();
	void OnExit_Idle();

	void OnEnter_Walk();
	void OnUpdate_Walk();
	void OnExit_Walk();
	void OnEnd_Walk();

	//L'update non obligatoire, peut avoir une utilité de debug
	virtual void Update() override;

	//Une fonction de test lié au systeme de frame event
	void TestFunction();
};
```
---
### .cpp:
```c
#include "AnimationController_Player.h"
#include <iostream>
#include <SFML/Window.hpp>

AnimationController_Player::AnimationController_Player(std::shared_ptr<Animator> _animator)	
: AnimationController(_animator){}

//Fonction d'initialisation obligatoire
void AnimationController_Player::InitAnimationController()
{
    //Procédure personnelle:
	InitAnimationStates();
	InitAndAssigneAnimations();

    //Utilisation de la fonctionnalité d'ajout de state
	AddStateToMachine(idleState);
	AddStateToMachine(walkState);

    //State de depart
	SetState(idleState);
}

void AnimationController_Player::InitAnimationStates()
{
    //Création des AnimationState. Une fonction static simplifiant la syntaxe
    //Le nom donné est important, il sert ensuite de clef pour la map qui contient les states
	idleState = AnimationState::CreateState("idle");
	walkState = AnimationState::CreateState("walk");

    //Assignation des fonctions
    //Afin que le systeme soit generique, les AnimationState sont dotées de variables
    //de type std::function<void()>.
    //On peut alors assigner n'importe quelle fonction de meme signature sur ces "slot"
	idleState->OnEnter = [this]() { this->OnEnter_Idle(); };
	idleState->OnExit = [this]() {this->OnExit_Idle(); };

	walkState->OnEnter = [this]() { this->OnEnter_Walk(); };
	walkState->OnExit = [this]() {this->OnExit_Walk(); };
	walkState->OnEnd = [this]() {this->OnEnd_Walk(); };
}

void AnimationController_Player::InitAndAssigneAnimations()
{
    //Création des animations
	auto idleAnimation = new Animation();
	idleAnimation->SetLooping(true);
    //Création des séquences droite et gauche de l'animation Idle
	auto idleRightSequence = idleAnimation->GenerateSequence(0, 0, 48, 48, 9);
	auto idleLeftSequence = idleAnimation->GenerateSequence(0, 48, 48, 48, 9);
    //Assignation des séquences à leur index
	idleAnimation->SetSequence(0, idleRightSequence);
	idleAnimation->SetSequence(1, idleLeftSequence);
    //Application de la séquence de départ
	idleAnimation->SetSequenceKey(0);
	idleAnimation->ActivateFirstSequence();
	idleAnimation->SetAnimationFrequency(0.05f);
    //Assignation de l'animation au state correspondant
	idleState->SetAnimation(idleAnimation);

    //Création d'une fonction de test,
    //Appliquée sur la frame 5 de l'animation idle.
	auto function = [this]() { TestFunction(); };
	idleAnimation->SetFrameEvent(5, function);

    //Meme procédure pour l'animation walk
	auto walkAnimation = new Animation();
	walkAnimation->SetLooping(true);
	auto walkRightSequence = walkAnimation->GenerateSequence(0, 48 * 2, 48, 48, 4);
	auto walkLeftSequence = walkAnimation->GenerateSequence(0, 48 * 3, 48, 48, 4);
	walkAnimation->SetSequence(0, walkRightSequence);
	walkAnimation->SetSequence(1, walkLeftSequence);
	walkAnimation->SetSequenceKey(0);
	walkAnimation->ActivateFirstSequence();
	walkAnimation->SetAnimationFrequency();
	walkState->SetAnimation(walkAnimation);
}

#pragma region WALK
void AnimationController_Player::OnEnter_Walk()
{
    //Il est important de lancer la fonction Enter d'un state, 
    //car c'est elle qui se charge de reset le compteur de frame et le timer de l'animation.
	walkState->Enter();
}

void AnimationController_Player::OnExit_Walk()
{
    //Cet appel n'est pas important pour l'instant
	walkState->Exit();
}

void AnimationController_Player::OnEnd_Walk()
{
    //Cet appel a End est important car il s'assure que l'animation est stoppée.
	walkState->End();
}

//Override de l'update inutile à moins de vouloir debug quelque chose.
void AnimationController_Player::Update()
{
	AnimationController::Update();
}
#pragma endregion

#pragma region Idle
void AnimationController_Player::OnEnter_Idle()
{
	idleState->Enter();
}

void AnimationController_Player::OnExit_Idle() {}
#pragma endregion

void AnimationController_Player::TestFunction()
{
	std::cout << "test event" << std::endl;
}
```