# GameLoop

La GameLoop, armée de sa seule fonction Run(Scene* _scene),  
se charge de faire tourner le programme.

```c
	void GameLoop::Run(Scene* _scene)
	{
		while (m_window->IsDone() == false)
		{
			//EVENT SECTION =====================================================================
			sf::Event event;
			while (m_window->PollEvent(event))
			{
				if ((event.type == sf::Event::Closed) || (event.type == sf::Event::KeyReleased && event.key.code == sf::Keyboard::Key::Escape))
					m_window->WindowDestroy();
				else
					_scene->HandleEvent(event, m_window->GetRW());
			}
			//===================================================================================

			//TEMPS
			TimeTool::RestartClock();
			m_keyTimer += timeDelta;
			sf::Joystick::update();

			if (sf::Keyboard::isKeyPressed(sf::Keyboard::F9) && m_keyTimer > 0.2f)
			{
				m_window->ToggleFullscreen();
				m_keyTimer = 0.f;
			}

			//Les ressources de la GameSection sont prete, donc on peut s'en servir
			if (_scene->IsReady())
			{
				_scene->Update(m_window->GetRW());

				m_window->BeginDraw();
				if (_scene->IsStopped() == false)
					_scene->Display(m_window->GetRW());
				m_window->EndDraw();
			}

			//Recupere l'éventuelle Scene préparée en m_nextScene de la Scene en cour
			Scene* nextScene = _scene->PullNextScene();
			if (nextScene != nullptr) //Si elle n'est pas null, alors on s'en sert pour remplacer l'actuelle
			{
				if (nextScene->IsInited() == false)
				{
					if (nextScene->UseRessourceManagerFromPreviousScene())
						nextScene->SetRessourceManager(_scene->GetRessourceManager());
				
					nextScene->Init();
				}
				delete _scene; //Cette destruction de la Scene se charge de détruire les ressources qui ne serviront plus.
				_scene = nextScene;
			}
			bool quitGame = _scene->PullQuitGame();
			if (quitGame)
			{
				m_window->GetRW().close();
			}
		}
		delete _scene; //Fenetre fermée, donc on détruit la Scene, et donc ses composants.
	}
}
```
