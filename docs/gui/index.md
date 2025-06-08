

Nostr Game Engine (NGE) includes a built-in UI module built on top of [Lemur](./lemur.md), a powerful GUI toolkit for jMonkeyEngine.


## Core Concept

The NGE GUI module is designed around a single active **window**. At any time, only one window is visible on the screen. When a new window is opened, it replaces the previous one, but provides a **back button** to return. 

This design keeps things simple and makes it easy to support touch devices and gamepads. If you need more complex behavior, you can mix and match standard [Lemur](./lemur.md) components with the NGE GUI module, or skip the NGE GUI module entirely and use Lemur directly.

A window can be fullscreen or small, transparent or opaque, centered or aligned differently, depending on your needs.

!!! example
    * A game HUD might use a fullscreen, transparent window.
    * A settings menu might be a smaller, centered window with an opaque background.

The system is built to cover the vast majority of game UI needs in a simple, scalable way.

!!! tip
    To avoid confusion with Java’s built-in UI classes, all NGE GUI components are prefixed with `N` (e.g., `NWindow`, `NButton`, etc.).



## Enabling the Window Manager


!!! abstract "See [NWindowManagerComponent Javadoc](https://javadoc.ngengine.org/org/ngengine/gui/win/NWindowManagerComponent.html)"

Everything in the UI system is managed by the `NWindowManagerComponent`. To use it, add and enable it like so (as shown in the [Getting Started section](../getting-started.md)):

```java
Runnable appBuilder = NGEApplication.createApp(settings, app -> {
    ComponentManager mng = app.getComponentManager();
    
    // Add the Window Manager Component
    mng.addAndEnableComponent(new NWindowManagerComponent());
    
    // ... add other components ...
});
```



## Creating a Window

!!! abstract "See [NWindow Javadoc](https://javadoc.ngengine.org/org/ngengine/gui/win/NWindow.html)"

To create a custom window, extend the `NWindow` class. Here's a simple example:

```java
public class MyWindow extends NWindow<Object> {
    @Override
    protected void compose(Vector3f size, Object args) throws Throwable {
        setTitle("My Window");

        NLabel label = new NLabel("A label");
        label.setTextHAlignment(HAlignment.Center);

        NButton confirmButton = new NButton("Confirm");
        confirmButton.addClickCommands(b -> {
            System.out.println("Confirmed!");
            close();
        });

        NColumn column = new NColumn();
        column.addChild(label);
        column.addChild(confirmButton);

        getContent().addChild(column);
    }
}
```

* The `compose()` method builds the window’s contents.
* This method may be called multiple times (e.g., when resizing or refreshing), but previous content is cleared automatically.
* Windows must not have constructors with arguments

!!! tip
    Think of this like an **immediate-mode UI**: you rebuild the content everytime the `compose()` method is called, but the engine maintains the state as long as possible

### Useful Window Methods

* `setTitle(String title)`: Set the window title.
* `getContent()`: Get the main content panel.
* `setFullScreen(boolean fullScreen)`: Make the window fullscreen (default: `false`).
* `setWithTitleBar(boolean withTitleBar)`: Show/hide the title bar (default: `true`).
* `setFitContent(boolean fitContent)`: Resize to fit content (default: `true`).
* `close()`: Close the window.


## Opening a Window

To display a window, use the `NWindowManagerComponent`:

```java
NWindowManagerComponent windowManager = mng.getComponent(NWindowManagerComponent.class);
windowManager.showWindow(MyWindow.class);
```

### With Callback:

```java
windowManager.showWindow(MyWindow.class, (window, err) -> {
    if (err != null) {
        System.err.println("Failed to open window: " + err.getMessage());
    } else {
        System.out.println("Window opened successfully: " + window);
    }
});
```

### With Arguments:

The `NWindow` class has a generic type for its argument that is passed to the `compose()` method. This allows you to pass data when opening a window, which can be useful for configuration or state management.

You can pass data like this:

```java
windowManager.showWindow(MyWindow.class, myArgs, optionalCallback);
```


## Closing a Window

To close a window, use the `close()` method, either from the instance or inside the window logic:

!!! example

    ```java
    NButton closeButton = new NButton("Close");
    closeButton.addClickCommands(b -> close());
    ```


## Toasts

Toasts are short-lived messages for feedback or notifications.
You can show a toast using the `NWindowManagerComponent`:

```java
windowManager.showToast(ToastType.INFO, "This is a toast message", Duration.ofSeconds(3));
```

You can also omit the duration and a default duration will be used depending on the toast type:

```java
windowManager.showToast(ToastType.INFO, "This is a toast message");
```

To display an exception:

```java
windowManager.showToast(new Exception("Something went wrong"));
```

This will automatically format and show the stack trace in a toast. It’s a good way to log and report non-blocking background errors to the user.



## Closing Everything

If you need to completely clear the UI (e.g., when starting gameplay):

```java
windowManager.closeAllWindows(); // Closes all windows
windowManager.closeAllToasts();  // Closes all toasts
windowManager.closeAll();        // Closes both
```


## Showing a Fatal Error

When something unrecoverable happens, show a fatal error message like this:

```java
windowManager.showFatalError(new Exception("This is a fatal error message"));
```

This will open a modal dialog with:

* A copy-to-clipboard button for the stack trace
* A button to exit the game

Perfect for critical crash handling and debugging.
