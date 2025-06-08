# Key Concepts

This page provides a high-level overview of the core concepts behind the **Nostr Game Engine (NGE)**.

NGE follows many of the same paradigms as [jMonkeyEngine 3 (jME3)](https://jmonkeyengine.org/), so if you're already familiar with jME3, you’ll feel right at home.

---

 


## Meshes


---

## ViewPorts



### Camera


## Assets



---

## Components and Controls

NGE introduces a flexible **component system** designed for modular and reusable game logic.

### Components

A **Component** is a modular unit of functionality attached to an app.
Components are composed of smaller fragments, and are managed by the `ComponentManager`. They integrate automatically with the engine systems.

### Controls

A **Control** is a behavior attached directly to a spatial. It is updated every frame and affects only the spatial it is attached to.

```java
spatial.addControl(new MyCustomControl());
```

!!! warning
    Controls are planned to be replaced in future releases with spatial-scoped components, which offer greater flexibility and cleaner separation of concerns.


---

## Input System

NGE supports various input methods:

* **Keyboard** — For commands and typing.
* **Mouse** — For pointing and clicking.
* **Touch** — For touchscreen devices.
* **Gamepad** — For controller-based input.

The input system is centered around the `InputManager` and can be accessed via an `InputHandlerFragment`.

---

## Audio Nodes


## DataStores 

A data store is key-value storage that can be used to store persistent data, such as player preferences, game state, or configuration settings.