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

The Components are registered to the Component System through a variadic template, a decision which I will discuss in the [Compiletime Component Registration an its Benefits](#compiletime-component-registration-an-its-benefits) section. The order of the Components in the registration will determine the update order of each Component. 
```cpp
	ComponentManager::GetInstance().RegisterComponents
	<
		Components::Transform,
		Components::ThirdPersonCamera,
		Components::PlayerController,
		Components::Flashlight,
		Components::WalkieTalkie,
		Components::Model,
		Components::Light,
		Components::Interactor,
		Components::Interactable,
		Components::AnimationController,
		Components::OBBCollider,
		Components::Trigger,
		Components::StaticEmitter,
		Components::EmitterSlotCollection,
		Components::BurstEmitter,
		Components::MeshEffectCollection,
		Components::StaticPhysicsBox,
		Components::StaticPhysicsTriangleMesh,
		Components::AudioComponent,
		Components::Pot,
		Components::GlobalParameterZone,
		Components::AudioSource,
		Components::DynamicRigidBody
	>();
```

The GameObjects which the Components are attached to, aren't updating the Components themselves, but the Components are rather updated by the Component System in collections of their own type. For instance, all of the currently active Transform Components will update before all of the active Model Components. This ensures us the update order control we need when developing various games. 


## GameObjects
As I described earlier, GameObjects are essentially a collection of Components. In our case, GameObjects are responsible for the attachment and detachment of Components to themselves, and much more.
```cpp
class Component;
class GameObject
{
	friend class GameObjectDebugger;

public:

	GameObject();
	~GameObject();

	template<class T>
	T*	   AttachComponent();
	Component* AttachComponent(std::string aComponent);

	template<class T>
	void DetachComponent();
	void DetachComponent(std::string aComponent);

	void DetachAllComponents();

	template<class T>
	T* GetComponent();

	void DisableAll();
	void EnableAll();

	void ShowComponentData();
	void HideComponentData();

	void KillMe();
	void ReviveMe();
	
	bool IsDead();
	
	inline const std::string& GetName() const { return myName; }	
	inline void  SetName(const std::string& aName) { myName = aName; }
	inline void	 SetID(UINT aID) { myID = aID; }
	inline UINT	 GetID() { return myID; }

	std::map<std::type_index, Component*>& GetComponents();

	void ClearComponentMap();

	//Clones all components to the out parameter
	void CloneTo(GameObject& aGameObjectOut);

	//Clones all components from the in parameter
	void CloneFrom(const GameObject& aGameObjectIn);
	
	void SetComponentAttachOrder(std::vector<std::type_index> aComponentAttachOrder);
	void SetIsBlueprint(bool aIsBlueprint);
	bool IsBlueprint() const;
```

Components can be attached through a templated function, but also with a std::string, which I will touch upon later in the [Compiletime Component Registration an its Benefits](#compiletime-component-registration-an-its-benefits) section! An Example of an attachment of a Component would be a player:
```cpp myPlayer->AttachComponent<Components::Transform>();```

The pointers to the Components attached to the GameObject are in this case saved in a std::map, with a std::type_index as its key. 
```cpp std::map<std::type_index, Component*> myComponents;```

This essentially means that you can only have one instance of a Component type attached to the GameObject, which differs from for instance Unity's Component System, where multiple instances of the same type can be attached. The reasoning behind this was to make the whole system iterable towards a ECS (Entity Component System), which we didn't really have time to do at the end. 


## The System



## Compiletime Component Registration an its Benefits
