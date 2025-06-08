
The PBR material system in NGE uses **Image-Based Lighting (IBL)** to enable advanced features like environment reflections, refractions, global illumination, and more.

This allows for highly realistic rendering of materials under various lighting conditions. However, it requires additional setup in the scene. Specifically, a **LightProbe** must be configured to provide the necessary lighting information. 

The LightProbe can be **pre-baked** or **dynamically generated** based on the scene’s environment.

!!! abstract "See [LightProbe Javadoc](https://javadoc.ngengine.org/com/jme3/light/LightProbe.html)"

In this guide, we'll cover the basic and most straightforward way to set up a PBR scene using a [Sky](../sky.md) or other supporting environment geometry, along with the [NGE Accelerated Baker](https://javadoc.ngengine.org/com/jme3/environment/baker/IBLGLEnvBakerLight.html) to generate the required LightProbe in near real-time.


## Step-by-Step Setup

### Attach a Sky (or Supporting Geometry) to the `rootNode`

This geometry provides the visual environment used during the LightProbe bake.

```java
Spatial sky = SkyFactory.createSky(assetManager, skyTexture, SkyFactory.EnvMapType.EquirectMap);
rootNode.attachChild(sky);
```




###  Add EnvironmentProbeControl to the Scene


This control manages the generation and assignment of the LightProbe for the node it's attached to and all its children.

```java
int resolution = 256; // Resolution of the environment probe
rootNode.addControl(new EnvironmentProbeControl(assetManager, resolution));
```



### Tag the Supporting Scene Geometry

To ensure the environment is used during LightProbe baking, tag it using `EnvironmentProbeControl.tagGlobal()`:

```java
EnvironmentProbeControl.tagGlobal(sky);
```



## That’s It!

On the first frame, the `EnvironmentProbeControl` will bake the LightProbe and apply it automatically to all PBR materials in the scene. 

No further setup is required: materials will pick up the probe lighting automatically.



## Advanced Usage
!!! abstract "See [EnvironmentProbeControl Javadoc](https://javadoc.ngengine.org/com/jme3/environment/EnvironmentProbeControl.html)"

You can use multiple `EnvironmentProbeControl` instances within a scene. Their generated LightProbes will blend together using their position and radius as weights.

To assign specific environment geometries to individual probes, use the instance method `tag(spatial)` instead of the global one:

```java
environmentProbeControl.tag(myCustomEnvironmentSpatial);
```

This gives you finer control over which parts of the scene contribute to each LightProbe.

