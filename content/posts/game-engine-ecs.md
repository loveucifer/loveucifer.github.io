---
title: "I've Been Writing a 2D Game Engine (ECS)"
date: 2026-07-16
draft: false
description: "Entity Component System — a better way to organize your game engine"
---

Onto the next part what we have is entity component system

so what is an entity component system , ecs for short , ecs is a way of handling multiple componenets in our engine , there are 2 ways to go about it usually we can either do inheritance , inheritance as for eg there might be a person and the person inherits specific features maybe like transform componenet , sprite componenet etc , inheritance is one of th 4 fundamental pillars of OOPs as well but we go about with entity componenet system model of organizing , we can think of inheritance method as It lets a class reuse and extend behavior from a parent class and it almost always causes I tight coupling between parent and child classes that is change the parent, and you can silently break every subclass which is not how we want our engine to work so we favour towards ecs

so how does it work ?

think about an entity whatever it maybe player , an enemey or just a car or whatever , each of them are individual entities of their own , for eg a player might need a transform component , but maybe something like a car door dosent need a transform componenet , but in the same way we can have a collider componenet for all the entieies that are enemy player and car , but they are all seperate entities of their own , so in a way we have entities that have specific componenets , so obviously we need a manger for this so we have an entitymanager managing the componenets of entities , i think that makes sense ( for me it does ).

so we can do something like

```cpp
class Entity {
    vector<component> components;
    add components<T> component
    updatethe component
    render the component
}

class EntityManager {
    vector <entity> entities;
    add entity(name of the entity)
    get entity(string entity name)
    update the entity
    render the entity
}

class component {
    entity owner. // tracking the owner of the component
    virtual update()
    virtual render()
}
```

ok so why do we have component update and renders as virtual well becasue they arent actually doing the update and render are they we should have some compenent maybe transfrom componenet where we call whatver we want to do with this componenet with update overrirde same way we can have a collidercomponenet that uses this virtual udpate and render and overrirde when we want it.

so in short

```
Game
  holds the entitymanager

    Entity manager
      hold the list of entities
      for each entity
        entity update and render

        Entity
          for each entity has a list of components
          for each component
            component update and render
```

explaining the template function to myself

```cpp
template <typename T, typename... TArgs>
T& AddComponent(TArgs&&... args) {
    T* newComponent(new T(std::forward<TArgs>(args)...));
    newComponent->owner = this;
    components.emplace_back(newComponent);
    newComponent->Initialize();
    return *newComponent;
}
```

we can think of this function working as follows "Create a component, attach it to this entity, initialize it, and give it back to me."

Step 1 is we do template <typename T, typename... TArgs>

t = type of componenet we want to add
Targs = the constructor argument for that component like int double or whatver

for eg entity.AddComponent<HealthComponent>(100);
here T is healthcomponenet the type and Targs is just int with a value of 100 in this case

Step 2: Create the component

T* newComponent(new T(std::forward<TArgs>(args)...));

this creates the component so if we called
entity.AddComponent<HealthComponent>(100);

it becomes something like
HealthComponent* newComponent = new HealthComponent(100);

Step 3: set as owner

step3 is making the component know its entity as each component should attatch to its entity

newComponent->owner = this;

Step 4: store it

the next step is to store whatver entity we have just created

components.emplace_back(newComponent);

this adds the new componenet to the entityts list of components

so in working it would be like
components = [Transform, Sprite]
and after adding it it would go from this to
components = [Transform, Sprite, Health]

the final step is to return it and we cant return the whole entity and risk making a copy
we can just return a pointer to this with
return *newComponent;

commit - https://github.com/loveucifer/eclipse/commit/5bcfb5c6aa2fd511dd22664994d382fedaa93ae2
