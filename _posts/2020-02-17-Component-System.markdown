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
- [Conclusion](#conclusion)

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
`myPlayer->AttachComponent<Components::Transform>();`

The pointers to the Components attached to the GameObject are in this case saved in a std::map, with a std::type_index as its key. 
`std::map<std::type_index, Component*> myComponents;`

This essentially means that you can only have one instance of a Component type attached to the GameObject, which differs from for instance Unity's Component System, where multiple instances of the same type can be attached. The reasoning behind this was to make the whole system iterable towards a ECS (Entity Component System), which we didn't really have time to do at the end. 


## The System
The Component System itself manages all the Components and its collections based on various situations. Components can be enabled or disabled (which describes whether their active or not) as the user wish (which makes them not update), which are handled through sperate collections. 
```cpp
void* myComponents[ARGS_COUNT] = { nullptr };
std::vector<Component*> myActiveComponents[ARGS_COUNT];
```
`myComponents` is essentially one giant memory block of all Components, while `myActiveComponents` includes a std::vector for each Component type, which stores pointers to the current active Components of that type. The way the active Components gets updated looks like this:
```cpp
for (int componentType = 0; componentType < ARGS_COUNT; componentType++)
	{
	for (myActiveComponentIndex[componentType] = myActiveComponents[componentType].size() - 1;
	     myActiveComponentIndex[componentType] >= 0; 
	   --myActiveComponentIndex[componentType] )
	{
		myActiveComponents[componentType][myActiveComponentIndex[componentType]]->Update(aDeltaTime);
	}
}
```

The actual update order of the Components it determined by the variadic template regristration.


## Compiletime Component Registration an its Benefits
As I mentioned earlier, all of the Components are registered at compiletime through a variadic templated class, which is the Component System itself:
```cpp
template<class ... Args>
class ComponentSystem : public CU::VariadicIndexer<Args ... >, public ComponentInterface
```

`CU::VariadicIndexer<Args ... >` is used to assign an index to each argument within the variadic template at compiletime (method found [here](https://stackoverflow.com/questions/30736242/how-can-i-get-the-index-of-a-type-in-a-variadic-class-template)). This is extremely useful, because it allows us to generate look-up tables through a fold-expression. For instance:
```cpp
(RegisterTypeLookup<Args>(),	...);
```
The `...` is a so called [fold expression](https://en.cppreference.com/w/cpp/language/fold), which essentially unpacks the expression for each argument in the variadic template. So in this case, `RegisterTypeLookup` gets pasted for each argument, while `Args` gets replaced with a type from the argument list.

```cpp
template<class T>
void RegisterTypeLookup()
{
	myTypeIndexLookupTable[std::type_index(typeid(T))] = V_INDEX(T);
		
	std::string name = typeid(T).name();
	name.erase(0, 18);

	myStringNameLookupTable[name] = V_INDEX(T);
	myIDNameLookupTable[V_INDEX(T)] = name;
}
```

`T` is in this case the Component in question, and `V_INDEX(T)` is a macro that gives us the relevant index to that type within the variadic template, which is based on the `CU::VariadicIndexer<Args ... >` that I descrived earlier. 

Why is this so useful then? It allows us to essentially find a type from a relevant index, ID, or even a string, which means that we can attach Components to a GameObject with just a string defining the Component type. 

This also means that we would have to branch in our fold expression to find the relevant Component, which is not as trivial as writing an `if` statement within the fold expression, since `if`'s are not expressions. But we can achieve the same behaviour through using a ternary operator, with two lambdas for each outcome:
```cpp
Component* RetreiveComponent(std::string aComponent)
{
	Component* retreivedComponent = nullptr;
	int componentIndex = myStringNameLookupTable[aComponent];

	(((std::type_index(typeid(Args)) == aComponent ?
		[&retreivedComponent, &componentIndex, this]
		{
			retreivedComponent = (Component*)(((CU::ObjectPool<Args, 			 			
							    MAX_GAMEOBJECT>*)myComponents[componentIndex]))->Retrieve();
		}() :
		[]
		{	
			// not the component type
		}()))
	, ...);

	return retreivedComponent;
}
```

This ternary gets unfolded for each Component, and have two lambda outcomes that returns void. The first one sets `retreivedComponent` if it has found the relevant Component type, while the other lambda does nothing. 

Another big benefit this method has, is that you can find a base class's derived class:
```cpp
void CloneSpecificComponent(std::type_index aComponentType, Component* aFrom, Component* aTo)
{
	(((std::type_index(typeid(Args)) == aComponentType ?
		[&aFrom, &aTo]
		{
			*((Args*)(aTo)) = *((Args*)(aFrom));
		}() 
			:
		[]
		{
			// not the component type
		}()))
	, ...);
}
```

This function is used from within another function in GameObject, which is used to clone a GameObject into another. The GameObject only have the base Component pointers to the Components themselve, while still having the `std::type_index` for those Components. Because of this, we need to find which the specific type the Component is, cast it to the right type, and execute the `=` operator on that Component. In the case of the Component in question having pointers to relevant data, the creator of that Component can simply just overload the `=` operator and write their own copy method. 

The biggest benefit all of this had, was that we could attach Components to GameObject through an identifying string. Since we had a blueprint system where other disciplines can define GameObjects through a .json file, we saved a lot of time on not double registering (acknowledge the Component type more than once in the codebase) Components. Example of a blueprint:
```json
{
  "id": 10,
  "tag": "Hazmat",
  "components": [
    {
      "type": "Transform",
      "data": {
        "x": 0,
        "y": 0,
        "z": 0,
        "rx": 0,
        "ry": 0,
        "rz": 0,
        "sx": 1,
        "sy": 1,
        "sz": 1
      }
    },
    {
      "type": "Model",
      "data": {
        "mesh": "Assets/Characters/Hazmat/CH_NPC_Hazmat_SK.fbx",
		"shouldRender": true
      }
    },
    {
      "type": "AnimationController",
      "data": {
      }
    },
    {
      "type": "EmitterSlotCollection",
      "data": {
			"slotcollection":"Particles/slots_hazmat.json"
      }
    }
  ]
}
```

The loading of `"data"` could be defined through the virtual function `virtual void InitDataFromJson(const rapidjson::Value& aData)`.


## Conclusion
Writing this Component System served as an useful experience in engine/backend development in general, where I encountered many specific issues while maintaining the goal of keeping the interface as clean and user-friendly as possible. It was also a useful experience in the sense of that I gained more knowledge within C++, and that it opened my eyes for usecases. 
