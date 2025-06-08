!!! abstract "See [Camera Javadoc](https://javadoc.ngengine.org/com/jme3/renderer/Camera.html)"

The **Camera** determines how the scene is viewed and rendered.

* Has a **position**, **direction**, and **view frustum**.
* Every application has at least one active camera.

!!! example
    ```java
    cam.setLocation(new Vector3f(0, 5, 10));
    cam.lookAt(Vector3f.ZERO, Vector3f.UNIT_Y);
    ```

