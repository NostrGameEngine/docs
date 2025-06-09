# Component System

Nostr Game Engine features a flexible component system designed for modularity and extensibility. At its core, it enables dynamic assembly of game or application logic from reusable "components" managed by a central `ComponentManager`.


## Components
!!! abstract "See [Component Javadoc](https://javadoc.ngengine.org/org/ngengine/components/Component.html)"

A `Component` is a reusable unit of behavior or logic. Components are defined by implementing the `org.ngengine.components.Component` interface.

- **Reusable**: Components can be attached and detached at runtime.  
- **Identifiable**: Each component has an identifier and may belong to a "slot" for mutual exclusion.  
- **Lifecycle**: Components react to being attached, enabled, disabled, or detached from a manager.  
- **Fragments**: Components can implement multiple fragment interfaces for specialized capabilities (e.g., rendering, input handling, asset loading). The `ComponentManager` automatically recognizes and integrates these fragments with engine subsystems.

```java
public interface Component<T> {
    // Lifecycle methods (onAttached, onEnable, onNudge, onDisable, onDetached)
    // Identification and slot handling
}
```

!!! note "For jME3 Developers"
    In NGE, **Components** replace the traditional jME3 `AppState` system.  
    They offer a more modular and flexible approach that eliminates the need to pass external references manually or retrieve them from the application instance.  


## Fragments
!!! abstract "See [Fragment Javadoc](https://javadoc.ngengine.org/org/ngengine/components/fragments/Fragment.html)"

Fragments are interfaces that extend a component’s functionality. By implementing one or more fragment interfaces (e.g., `MainViewPortFragment`, `LogicFragment`), a component signals which engine systems it needs. The manager detects these fragments and handles their lifecycle in a predefined order:

1. **Initialization** (`receiveXXX` methods)  
2. **Loading** (`loadXXX` methods)  
3. **Enable** (via the component’s `onEnable`)  
4. **Update** (`updateXXX` methods called on each update tick)  
5. **Disable** (via the component’s `onDisable`)

This composition-based design avoids deep inheritance hierarchies. Each fragment represents hooks into specific engine subsystems, allowing the component to participate in various aspects of the game/application lifecycle without being tightly coupled to the engine.



## ComponentManager
!!! abstract "See [ComponentManager Javadoc](https://javadoc.ngengine.org/org/ngengine/components/ComponentManager.html)"
The main orchestrator of components is the `ComponentManager`, that manages the lifecycle, dependencies, and interactions of components:


```java
ComponentManager manager = ...;
manager.addComponent(myComponent, dependency1, dependency2);
manager.removeComponent(myComponent);
manager.enableComponent(myComponent, optionalArgument);
manager.disableComponent(myComponent);
```

See [Getting Started](../getting-started.md#using-components-from-components-and-dependencies) to learn how to connect components together.




## Development Workflow

You should start by creating a class that implements the `Component` interface.  


!!! example
    ```java
    public class MyComponent implements Component<Object> {
        // Implementation...
    }
    ```

from there you should proceed to implement as many fragments as needed, until you hook into all the necessary engine subsystems and have access to the objects you need for your component to work.

!!! example
    ```java
    // A component that listen for an input and draw a sprite will implement
    public class FireballComponent implements Component<Object>, InputFragment, AssetLoadingFragment {
        // Implement methods for input handling and rendering
    }
    ```


A list of available fragments can be found in the [Javadoc](https://javadoc.ngengine.org/org/ngengine/components/fragments/package-summary.html).