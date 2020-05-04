---
layout: post
title: Component System
permalink: /Component System/
description: A Unity-esque Component System 
onhome: false
---
## Contents
- [Background](#background)
- [The Structure](#the-structure)
- [Components](#components)
- [GameObjects](#gameobjects)
- [The System](#the-system)
- [Compiletime Component Registration an its Benefits](#compiletime-component-registration-an-its-benefits)


## Background
I've always loved to try out new ways of handling entities in games, which has lead to a lot of experiments over the years. So when the programmers of Oddbox were deciding the engine structure, I got the task os creating a Component System for our engine. 

The one thing I wanted to focus on was the interface itself for the Component System. I wanted it to be as easy to use as possible, so that we could save as much time as possible, and minimize any confusion in how the Component System is supposed to be used. 


## The Structure
The structure is pretty simple at a surface level. GameObject are essentially the in-game actors who controls objects in a given seen, which can include AI, the player, or even a static object such as a rock. A GameObject is defined as a collection of "Components". 

Components themselves are defined as data with certain logic that can be invoked on that data. An example would be a car GameObject, which can for instance have a wheel Component attached to it. The wheel component is responsible for the steering of the car, and acts upon it. 


## Components
So how does this translate to code? I decided to make Component a base class, which newly created Components can inherit from. 

```cpp
class Component
{
	template<class ... >
	friend class ComponentSystem;

	friend class GameObject;
	friend class GameObjectDebugger;

public:

	Component();
	~Component();

	virtual bool Update(float aDeltaTime) = 0;
	virtual void LoadFileValues();

	template<class T>
	T* GetComponent();

	void Enable();
	void Disable();

	std::string& GetType();
	
	void SetHost(GameObject* aHost);
	GameObject* GetHost() { return myHost; }

	bool IsEnabled() const;
	
	virtual void OnAttach() = 0;
	virtual void OnDetach() = 0;
	virtual void OnKillMe() = 0;

	virtual void InitDataFromJson(const rapidjson::Value& aData);
```
( Just included the interface for simplicity ) 

Every newly created Component which inherits from Component will need to define its logic, which can both act upon its own data, or interface with some other attached Component to it's GameObject host. This can be done with the templatet function `GetComponent`, which is defined as such: 

```cpp
template<class T>
inline T* Component::GetComponent()
{
	return myHost->GetComponent<T>();
}
```

An exampe of an usecase, would be our Model Component, which pushes a render command in its defined logic.
```cpp
			GI::PushRenderCommand( myMeshHandle.GetID(), myAnimationControllerID, myShaderInstanceID, GetComponent<Transform>()->GetPosition(), GetComponent<Transform>()->GetForward(), GetComponent<Transform>()->GetUp(), GetComponent<Transform>()->GetScale(), myColor );
```



## GameObjects


## The System


## Compiletime Component Registration an its Benefits
