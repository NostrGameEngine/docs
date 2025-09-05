!!! info "From jME3 wiki"
    This is an excerpt from the jME3 wiki, check the original articles for more details: [Light and Shadow](https://wiki.jmonkeyengine.org/docs/3.4/core/light/light_and_shadow.html)

You can add several types of light sources to a spatial using 

`sp.addLight(mylight);`

The available light sources in [com.​jme3.​light](https://javadoc.ngengine.org/com/jme3/light/package-summary.html) are:

- SpotLight
- PointLight
- AmbientLight
- DirectionalLight
- LightProbe

You control the color and intensity of each light source. Typically you set the color to white ( `new ColorRGBA(1.0f,1.0f,1.0f,1.0f` or `ColorRGBA.White`) , which makes elements appear in their natural color.

You can choose to use lights in other colors than white, or darker colors. This influences the scene’s atmosphere and will make the scene appear colder (e.g. `ColorRGBA.Cyan`) or warmer (`ColorRGBA.Yellow`), brighter (higher values) or darker (lower values).


Lights attached to a spatial will illuminate only the spatial itself and its children. If you want to illuminate the entire scene, you should initialize them as **global** lights by passing `true` as `global` parameter to the light constructor.

```java
PointLight globalLight = new PointLight(true);
```

You can get a list of all lights added to a Spatial by `calling getWorldLightList()` (includes inherited lights and global lights) or `getLocalLightList()` (only directly added lights), and iterating over the result.


## PointLight
!!! abstract "See [PointLight Javadoc](https://javadoc.ngengine.org/com/jme3/light/PointLight.html)"


A PointLight has a location and shines from there in all directions as far as its radius reaches. The light intensity decreases with increased distance from the light source.


!!! example "Example: Lamp, lightbulb, torch, candle"

```java
PointLight lamp_light = new PointLight(true); // global light
lamp_light.setColor(ColorRGBA.Yellow);
lamp_light.setRadius(4f);
lamp_light.setPosition(new Vector3f(lamp_geo.getLocalTranslation()));
rootNode.addLight(lamp_light);
```

## DirectionalLight
!!! abstract "See [DirectionalLight Javadoc](https://javadoc.ngengine.org/com/jme3/light/DirectionalLight.html)"

A house model illuminated with a sun-like directional light

A DirectionalLight has no position, only a direction. It sends out parallel beams of light and is considered “infinitely” far away. You typically have one directional light per scene. 


!!! example "Example: Sun light"

```java
DirectionalLight sun = new DirectionalLight(true); // global light
sun.setColor(ColorRGBA.White);
sun.setDirection(new Vector3f(-.5f,-.5f,-.5f).normalizeLocal());
rootNode.addLight(sun);
```


## SpotLight

!!! abstract "See [SpotLight Javadoc](https://javadoc.ngengine.org/com/jme3/light/SpotLight.html)"

A SpotLight sends out a distinct beam or cone of light. A SpotLight has a direction, a position, distance (range) and two angles. The inner angle is the central maximum of the light cone, the outer angle the edge of the light cone. Everything outside the light cone’s angles is not affected by the light.

!!! example "Example: Flashlight"

```java
SpotLight spot = new SpotLight(true); // global light
spot.setSpotRange(100f);                           // distance
spot.setSpotInnerAngle(15f * FastMath.DEG_TO_RAD); // inner light cone (central beam)
spot.setSpotOuterAngle(35f * FastMath.DEG_TO_RAD); // outer light cone (edge of the light)
spot.setColor(ColorRGBA.White.mult(1.3f));         // light color
spot.setPosition(cam.getLocation());               // shine from camera loc
spot.setDirection(cam.getDirection());             // shine forward from camera loc
rootNode.addLight(spot)
```

## AmbientLight

!!! abstract "See [AmbientLight Javadoc](https://javadoc.ngengine.org/com/jme3/light/AmbientLight.html)"

An AmbientLight simply influences the brightness and color of the scene globally. It has no direction and no location and shines equally everywhere. An AmbientLight lights all sides of Geometries evenly, which makes 3D objects look unnaturally flat; this is why you typically do not use an AmbientLight alone without one of the other lights.

!!! example "Example: Regulate overall brightness, tinge the whole scene in a warm or cold color"

```java
AmbientLight al = new AmbientLight(); // local light
al.setColor(ColorRGBA.White.mult(1.3f));
rootNode.addLight(al);
```


## LightProbe
!!! abstract "See [LightProbe Javadoc](https://javadoc.ngengine.org/com/jme3/light/LightProbe.html)"
A LightProbe is a special light source that captures the light in a specific area of the scene and applies it to the objects within that area. It is used by the PBR Material to simulate realistic lighting effects.

See [Using PBR Lighting](../materials/index.md#using-pbr-lighting) for more details.

---


!!! tip
    You can increase the brightness of a light source gradually by multiplying the light color to values greater than 1.0f.
    Example: `mylight.setColor(ColorRGBA.White.mult(1.3f)`);


---

!!! tip 
    You can make a light follow a spatial by updating the light's position in a [Component](./components/index.md) or [Control](./controls.md)

--- 
    
    

## Shadows

Lights won't produce shadows by their own, for that you need to add shadow filters to the [Post processing pipeline](../postprocessing/index.md#shadows)