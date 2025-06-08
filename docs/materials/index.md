
!!! info "From jME3 wiki"
    This is an excerpt from the jME3 wiki, check the original articles for more details: [How to Use Materials](https://wiki.jmonkeyengine.org/docs/3.4/core/material/how_to_use_materials.html)

Materials determine how geometries are rendered.
Each material is based on a **material definition** (J3MD file), which defines the rendering logic and shader parameters.

!!! abstract "See [Material Javadoc](https://javadoc.ngengine.org/com/jme3/material/Material.html)"

J3MD files abstract shader configuration, providing a simple interface to set visual properties.

!!! tip
    J3MD uses a domain-specific language designed for jME3. Refer to the [jME3 Material Specification](https://wiki.jmonkeyengine.org/docs/3.4/core/material/material_specification.html) for more details.

Advanced users can define custom material definitions, while common ones are included in the engine.

!!! example

    ```java
    Material mat = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");
    mat.setColor("Color", ColorRGBA.Blue);
    myGeometry.setMaterial(mat);
    ```

A list of built-in material definitions can be found at the end of this page.

!!! info "For jME3 developers"
    J3M files are supported for backward compatibility but are considered deprecated.
    Prefer using the `Material` class directly in code.

---

## Transparency

Most materials support an alpha channel to make a model transparent, this can be achieved by setting a color or texture with an alpha channel.
However by default NGE uses front-to-back rendering for performance reasons, which means that transparent objects are not rendered correctly, to fix this you need to set the `Transparent` queue bucket for the `Spatial` and an appropriate blend mode in the material's additional render state.

See example below:

!!! example "Common example of a transparent spatial"
    ```java
    Material mat = new Material(assetManager, "Common/MatDefs/Misc/Unshaded.j3md");
    mat.setColor("Color", new ColorRGBA(1, 1, 1, 0.5f)); // Set color with alpha
    mat.getAdditionalRenderState().setBlendMode(BlendMode.AlphaSumA);

    myGeometry.setMaterial(mat);    
    myGeometry.setQueueBucket(Bucket.Transparent);
    ```


## Shaders

Materials use shaders written in GLSL, check the [jME3 Shader Documentation](https://wiki.jmonkeyengine.org/docs/3.4/core/shader/jme3_shaders.html) for more details.

When writing shaders for NGE you should generally target **OpenGL ES 3.0** and **OpenGL 3.2** or **OpenGL 3.3** as they guarantee compatibility with most devices and platforms.


!!! note "For jME3 developers"
    NGE is backward compatible with jME3, but you should assume it will always run on OpenGL ES 3.0 compatible hardware in core profile, everything below that is considered legacy and deprecated.

## Default Materials

!!! abstract "See [Materials Javadoc](https://javadoc.ngengine.org/com/jme3/material/Materials.html)"

NGE comes with several built-in material definitions that you can use directly in your projects.
The most commonly used material definitions are:

- **Materials.UNSHADED or "Common/MatDefs/Misc/Unshaded.j3md"** 
    - This material does not perform any lighting calculations and is suitable for simple objects or UI elements.
- **Materials.PBR or "Common/MatDefs/Light/PBRLighting.j3md"** 
    - This material implements physically-based rendering (PBR) techniques, providing realistic lighting and shading effects. It is ideal for high-quality materials and complex scenes.
    - The PBR material needs some extra setup in the scene, check [Setting Up a PBR Scene](./pbr.md) for more details.
- **Materials.LIGHTING or "Common/MatDefs/Light/Lighting.j3md"** 
    - This material supports basic lighting calculations, including ambient, diffuse, and specular lighting. It is an old lighting technique, if resources are available, prefer using PBR.
