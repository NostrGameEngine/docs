!!! abstract "See [ViewPort Javadoc](https://javadoc.ngengine.org/com/jme3/renderer/ViewPort.html)"

A **ViewPort** represents a rendering surface where one or more scenes (root nodes) are rendered.

!!! example
    ```java
    viewPort.attachScene(rootNode);
    Node scene = viewPort.getScenes().get(0);
    ```

Each application typically has:

* A **main viewport** — for the primary 3D scene.
* A **GUI viewport** — for 2D UI elements.

ViewPorts consist of:

* **Camera** — Defines the viewpoint.
* **Scenes** — Root nodes attached to the ViewPort.
* **Framebuffer** — Target render surface (typically the screen).
* **Render settings** — Clean flags, background color, etc.

!!! note
    A ViewPort can render multiple scenes simultaneously.

!!! tip
    You can create additional ViewPorts for advanced use cases like:

    - Mini-maps
    - Split-screen views
    - Off-screen rendering

    ```java
    ViewPort vp = renderManager.createMainView("MyView", camera);
    ```