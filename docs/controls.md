As discussed in the [Getting Started](./getting-started.md) section, NGE primarily uses **components** to encapsulate behavior. However, in some cases, you may want to attach logic directly to a **Spatial**. 

This is where **Controls** come in.


## What Is a Control?
!!! abstract "See [Control Javadoc](https://javadoc.ngengine.org/com/jme3/scene/control/Control.html)"

A **Control** is a behavior object that is attached directly to a spatial. It is updated every frame and should affect only the spatial it is attached to.

```java
spatial.addControl(new MyCustomControl());
```

!!! warning "Deprecation Notice"
    Controls are planned to be deprecated in future releases and replaced by spatial-scoped components, which offer better flexibility and a cleaner separation of concerns.

## Creating a Custom Control

To create your own control, extend the `AbstractControl` class and override its key lifecycle methods.

!!! example
    ```java
    public class MyControl extends AbstractControl {

        public MyControl() {
            super();
            // Initialization logic
        }

        @Override
        protected void controlUpdate(float tpf) {
            // Called every frame: implement per-frame behavior here
        }

        @Override
        protected void controlRender(RenderManager rm, ViewPort vp) {
            // Called just before rendering: implement custom rendering logic here
        }

        @Override
        public void write(JmeExporter e) throws IOException {
            super.write(e);
            // Serialize control state
        }

        @Override
        public void read(JmeImporter importer) throws IOException {
            super.read(importer);
            // Deserialize control state
        }
    }
    ```

NGE controls implement the `Savable` interface, allowing them to be saved as part of the scene graph. This means you should implement the `write` and `read` methods to correctly serialize and deserialize your control’s state.

Use the `OutputCapsule` and `InputCapsule` to handle your control’s fields.

!!! example
    ```java
    @Override
    public void write(JmeExporter e) throws IOException {
        super.write(e);
        OutputCapsule capsule = e.getCapsule(this);
        capsule.write(var1, "testSavable", null);
        capsule.write(var2, 123, null);
        capsule.write(var3, "testSavableWithDefault", Vector3f.ZERO);
    }

    @Override
    public void read(JmeImporter importer) throws IOException {
        super.read(importer);
        InputCapsule capsule = importer.getCapsule(this);
        var1 = (Matrix3f) capsule.readSavable("testSavable", null);
        var2 = capsule.readInt("testSavableWithDefault", 123);
        var3 = (Vector3f) capsule.readSavable("testSavableWithDefault", Vector3f.ZERO);
    }
    ```