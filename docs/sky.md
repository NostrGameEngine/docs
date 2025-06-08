NGE can use ldr and hdr images to render the sky and simulate ambient lighting in a scene.

!!! abstract "See [SkyFactory Javadoc](https://javadoc.ngengine.org/com/jme3/util/SkyFactory.html)"

## Sky Boxes

Sky boxes can be loaded from a cube map

```java
SkyFactory.createSky(assetManager, "path/of/cubemap-texture.dds", EnvMapType.CubeMap);
```

or from a set of 6 images:

```java
Texture west = assetManager.loadTexture("path/to/posx.jpg");
Texture east = assetManager.loadTexture("path/to/negx.jpg");
Texture north = assetManager.loadTexture("path/to/posz.jpg");
Texture south = assetManager.loadTexture("path/to/negz.jpg");
Texture up = assetManager.loadTexture("path/to/posy.jpg");
Texture down = assetManager.loadTexture("path/to/negy.jpg");

SkyFactory.createSky(assetManager, west, east, north, south, up, down);
```


## Equirectangular Skies (recommended)

NGE supports equirectangular skies, which are a single panoramic image that can be used to create a skybox. 
They are often used for HDR skies, they are easier to create, widely available, provide a better quality result in some cases, but are less performant than cube maps.

They are the preferred choice expecially for HDR skies [you can find many free ones online](https://polyhaven.com/hdris).

```java
SkyFactory.createSky(assetManager, "path/of/sky-texture.hdr", EnvMapType.EquirectMap);
```

you can also use a PNG or JPG image, but it will not be HDR and will not provide the same quality as a HDR image.


!!! tip
    When using an hdr sky you need to set the the [Tone Map Filter](../postprocessing/#tonemap) to get the best results, otherwise the sky will look too bright or washed out.
