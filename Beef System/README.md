# Beef System

My master thesis is divided into two parts

1. Get a fundamental understanding of code by learning SDL2 and C++ 
2. Create a small scale, [nemesis like system](https://www.polygon.com/middle-earth-shadow-of-war-guide/2017/10/9/16439610/the-nemesis-system-and-you) 

## Part 1
To take some of the black box feeling away from game programming I've decided to forego any engines and learn C++ by using [SDL2](https://www.libsdl.org/). One of the challenges by doing that is that I have to completely recreate what's usually just double clicking the unity icon. 

But it's fun!

So far I've done a `game loop`, `update method` and a `entity component system` and you can check out all that code in the [main repository](https://github.com/emilxf-0/beef-system), but what I wanted to showcase from this first part is the `Entity Component System` and how I use it to create my game world. 

## Entity Component System

### How It Works:

1. **Entity Creation:**
    - Entities are created using the `addEntity` function of the `Manager` class.
    - Each entity has a collection of components (stored in the `components` vector).
2. **Component Management:**
    - Components are added to an entity using the `addComponent` function. This function creates a new component, associates it with the entity, and updates the bitset and array accordingly.
3. **Update and Draw:**
    - The `update` and `draw` functions of the `Entity` class iterate through the components of an entity and call their respective `update` and `draw` functions.
4. **Refresh:**
    - The `refresh` function of the `Manager` class removes inactive entities from the `entities` vector. Entities can be deactivated using the `destroy` function.

```c++
class Component;
class Entity;

using ComponentID = std::size_t;

inline ComponentID getComponentTypeID()
{
	static ComponentID lastID = 0;
	return lastID++;
}

template <typename T> inline ComponentID getComponentTypeID() noexcept
{
	static ComponentID typeID = getComponentTypeID();
	return typeID;
}

constexpr std::size_t maxComponents = 32;

using ComponentBitSet = std::bitset<maxComponents>;
using ComponentArray = std::array<Component*, maxComponents>;

class Component
{
public:
	Entity* entity;

	virtual void init() {}
	virtual void update() {}
	virtual void draw(float interpolation) {}

	virtual ~Component() {}
};

class Entity
{
public:
	void update()
	{
		for (auto& component : components) component->update();
	}

	void draw(float interpolation)
	{
		for (auto& component : components) component->draw(interpolation);
	}
	bool isActive() const { return active; }
	void destroy() { active = false; }

	template <typename T> bool hasComponent() const
	{
		return componentBitSet[getComponentTypeID<T>()];
	}

	template <typename T, typename... TArgs >
	T& addComponent(TArgs&&... mArgs)
	{
		T* component(new T(std::forward<TArgs>(mArgs)...));
		component->entity = this;
		std::unique_ptr<Component> unique_ptr{ component };
		components.emplace_back(std::move(unique_ptr));

		componentArray[getComponentTypeID<T>()] = component;
		componentBitSet[getComponentTypeID<T>()] = true;

		component->init();
		return *component;
	}

	template<typename T> T& getComponent() const
	{
		auto ptr(componentArray[getComponentTypeID<T>()]);
		return *static_cast<T*>(ptr);
	}
	

private:
	bool active = true;
	std::vector<std::unique_ptr<Component>> components;

	ComponentArray componentArray;
	ComponentBitSet componentBitSet;

	
};

class Manager
{
private: std::vector<std::unique_ptr<Entity>> entities;

public:
	void update()
	{
		for (auto& entity : entities) entity->update();
	}
	void draw(float interpolation)
	{
		for (auto& entity : entities) entity->draw(interpolation);
	}

	void refresh()
	{
		entities.erase(std::remove_if(std::begin(entities), std::end(entities),
			[](const std::unique_ptr<Entity>& mEntity)
			{
				return !mEntity->isActive();
			}),
			std::end(entities));
	}

	Entity& addEntity()
	{
		Entity* entity = new Entity();
		std::unique_ptr<Entity> unique_ptr{entity};
		entities.emplace_back(std::move(unique_ptr));
		return *entity;

	}

	template <typename EntityType, typename... Args>
	EntityType& addEntity(Args&&... args)
	{
		EntityType* entity = new EntityType(std::forward<Args>(args)...);
		std::unique_ptr<Entity> unique_ptr{ entity };
		entities.emplace_back(std::move(unique_ptr));
		return *entity;
	}

};

```

## The components

Of course, the ECS doesn't do anything by itself it's just a framework. In order to create 
our game we first need to create components. 

Traditionally a player character need the following: a `transform` to determine position in the world, a `sprite` so we have a visual representation and a `controller` so we can move the player around in the world. Let's start from the top

### Transform component

```c++
#pragma once

#include "Components.h"
#include "Vector2D.h"

class TransformComponent : public Component
{

public:

	Vector2D position;
	Vector2D velocity;
	Vector2D lastPosition;

	int height = 32;
	int width = 32;
	int scale = 1;

	int speed = 3;


	TransformComponent()
	{
		position.x = 0.0f;
		position.y = 0.0f;
	}


	TransformComponent(float x, float y)
	{
		position.x = x;
		position.y = y;
	}

	TransformComponent(float x, float y, int h, int w, int s)
	{
		position.x = x;
		position.y = y;
		height = h;
		width = w;
		scale = s;
	}

	void init() override
	{
		velocity.x = 0;
		velocity.y = 0;
	}

	void update() override
	{
		lastPosition = position;
		position.x += velocity.x;
		position.y += velocity.y;
	}

};
```

### Sprite Component

```c++
#pragma once

#include "Components.h"
#include "Game.h"
#include "RotationComponent.h"
#include "SDL.h"
#include "TextureManager.h"

class SpriteComponent : public Component
{
private:
	TransformComponent* transform;
	SDL_Texture* texture;
	SDL_Rect srcRect, destRect;

public:
	RotationComponent* rotation;

	SpriteComponent() = default;

	SpriteComponent(const char* path)
	{
		setTexture(path);
	}

	~SpriteComponent()
	{
		SDL_DestroyTexture(texture);
	}

	void setTexture(const char* path)
	{
		texture = TextureManager::LoadTexture(path);
	}

	void init() override
	{
		transform = &entity->getComponent<TransformComponent>();

		if (!entity->hasComponent<RotationComponent>())
		{
			entity->addComponent<RotationComponent>();
		}
		rotation = &entity->getComponent<RotationComponent>();

		srcRect.x = srcRect.y = 0;
		srcRect.w = transform->width;
		srcRect.h = transform->height;
	}

	void update() override
	{
		destRect.x = static_cast<int>(transform->position.x);
		destRect.y = static_cast<int>(transform->position.y);
		destRect.w = transform->width * transform->scale;
		destRect.h = transform->height * transform->scale;
	}

	void draw(float interpolation) override
	{
		int interpolatedX = static_cast<int>(transform->position.x * (1.0f - interpolation) +
			(transform->lastPosition.x * interpolation));
		int interpolatedY = static_cast<int>(transform->position.y * (1.0f - interpolation) +
			(transform->lastPosition.y * interpolation));

		destRect.x = interpolatedX;
		destRect.y = interpolatedY;

		TextureManager::Draw(texture, srcRect, destRect, rotation->angle, SDL_FLIP_NONE);
	}
};

```

### Controller

```c++
#pragma once
#include "Game.h"
#include "EntityComponentSystem.h"
#include "Components.h"
#include "Input.h"
#include "Vector2D.h"

class Controller : public Component
{
public:
	TransformComponent* transform;
	RotationComponent* rotation;
	
	const Uint8* keyBoardState = SDL_GetKeyboardState(nullptr);

	Vector2D direction = Vector2D(0, 0);
	double angleInRadians = 0.0;

	float const SPEED = 3;
	float const ANGLE_INCREMENT = 1.0f;

	enum drivingState
	{
		IDLE,
		FORWARD,
		REVERSE,
		};

	drivingState drivingDirection;

	void init() override
	{
		transform = &entity->getComponent<TransformComponent>();
		rotation = &entity->getComponent<RotationComponent>();
		drivingDirection = IDLE;
	}

	void update() override
	{

		updateDirection();
		updateRotation();
		updateVelocity();
	}

private:
	void updateDirection()
	{
		angleInRadians = rotation->angle * (M_PI / 180);

		direction.x = SDL_cos(angleInRadians);
		direction.y = SDL_sin(angleInRadians);
		direction.normalize();

		if (keyBoardState[SDL_SCANCODE_W])
		{
			drivingDirection = FORWARD;
		}
		else if (keyBoardState[SDL_SCANCODE_S])
		{
			drivingDirection = REVERSE;
		}
		else
		{
			drivingDirection = IDLE;
		}
		
	}

	void updateRotation()
	{
		if (keyBoardState[SDL_SCANCODE_A])
		{
			rotation->angle -= ANGLE_INCREMENT;
		}
		else if (keyBoardState[SDL_SCANCODE_D])
		{
			rotation->angle += ANGLE_INCREMENT;
		}
	}

	void updateVelocity()
	{

		switch (drivingDirection)
		{
		case IDLE:
			transform->velocity.x = 0;
			transform->velocity.y = 0;
			break;

		case FORWARD:
			transform->velocity.x = direction.x * SPEED;
			transform->velocity.y = direction.y * SPEED;
			break;

		case REVERSE:
			transform->velocity.x = -direction.x * SPEED;
			transform->velocity.y = -direction.y * SPEED;
			break;

		default:
			break;
		}
	
	}

};

```


## Creating a Player

With all that set up we can easily create Entities that comprise of the components we like. 

In the Game.cpp we add a `player` entity and then we add all the components we like to that entity. 

```c++

auto& player(manager.addEntity());

// ...other code

player.addComponent<TransformComponent>();
player.addComponent<RotationComponent>();
player.addComponent<SpriteComponent>("assets/cars/player_car.png");
player.addComponent<Controller>();
player.addComponent<ColliderComponent>("player");

```
*et voila*
![](https://github.com/emilxf-0/Portfolio/blob/main/Beef%20System/AddPlayer.gif)

But that's not all. We can use the same system to add other entities as well. Let's say that we want to add a traffic light. 
```c++

auto& player(manager.addEntity());
auto& trafficLight(manager.addEntity());

// ...player code

trafficLight.addComponent<TransformComponent>(150, 150);
trafficLight.addComponent<SpriteComponent>("assets/props/trafficlight/red.png");

```
*Look it's a state machine*
![](https://github.com/emilxf-0/Portfolio/blob/main/Beef%20System/AddTrafficlight.gif)

And enemies

```c++
auto& enemy(manager.addEntity());

// ...player code
// ...traffic light code

enemy.addComponent<TransformComponent>();
enemy.addComponent<RotationComponent>();
enemy.addComponent<SpriteComponent>("assets/cars/enemy_car.png");
enemy.addComponent<ColliderComponent>("enemy");

```
*a wild road rager appears*
![](https://github.com/emilxf-0/Portfolio/blob/main/Beef%20System/AddTrafficlight.gif)


## What's next?

Now I'm at a point where I feel like I can start with the nemesis system. There are a lot of things I *could* do with this, like making a `AIcontroller` that I can pass into the controller component and make everything even more modular. But what I want to focus on now is creating interesting enemies that will have life of their own.   
