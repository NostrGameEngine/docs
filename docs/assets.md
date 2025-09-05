Assets are resource that can be loaded into the application at runtime using an `AssetManager`.

Assets include external resources such as:

* Models
* Textures
* Audio files
* Materials
* Fonts

These are typically loaded using the `AssetManager`, often within a [Component](./components/index.md) or a [Control](./controls.md).

The **AssetManager** can be extended with modular asset loaders to support additional formats. You can also attach asset locators to find assets in custom locations, such as the filesystem, URLs, or ZIP archives.

!!! abstract "See [AssetManager Javadoc](https://javadoc.ngengine.org/com/jme3/asset/AssetManager.html)"

!!! note
    By default, the `AssetManager` will look for assets in the resources of the classpath, typically under `src/main/resources` in a Maven project, this is usually good enought as it will make distribution easier.


```java
Spatial model = assetManager.loadModel("Models/Scene.j3o");
```

## Caching
The `AssetManager` will make reasonable efforts to cache assets in memory, so that if they are requested again, they can be retrieved quickly without reloading them from disk or network.

## Supported formats

NGE includes several loaders for backward compatibility with jME3, however, only the following subset is guaranteed to be officially supported in the future:

### Textures
* PNG
* JPG
* DDS: 2d textures and cubemaps, uncompressed or compressed in DXT1, DXT3, DXT5, RGTC1, RGTC2 or ETC2 formats
* HDR
* SVG
* WEBP

### Audio
* OGG Vorbis

### Models
* glTF 2.0
* Wavefront OBJ

glTF is the best and recommended way to get quality models into your app.
Models loaded in glTF that use the PBR material, should closely match the original model once imported, including textures and animations.

## Creating and importing 3D models

To create 3D models for NGE, you can use any software that exports to glTF 2.0.
We recommend using [Blender](https://www.blender.org/), which is a free and open-source 3D modeling software that supports glTF export natively.

Every model that uses the metal/rough PBR workflow should import pretty much as it is from Blender, including textures and animations.

!!! tip
    Make sure to configure the exporter corretly and to read the [Blender glTF docs](https://docs.blender.org/manual/en/2.80/addons/io_scene_gltf2.html) for best results.

The scene graph imported from a glTF model will closely match what you see in Blender, including the hierarchy of nodes, their names and their `Custom Properties` that are imported as `UserData` in NGE.

!!! tip
    You can use User Data to configure your scene behavior from blender itself, for example, you can set a `Custom Property` key called `isPlayer` to `true` on your player model, and then check for it in your components or controls to apply specific logic.
