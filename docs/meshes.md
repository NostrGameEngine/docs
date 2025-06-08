

!!! info "From jME3 wiki"
    This is an excerpt from the jME3 wiki, check the original articles for more details: [Mesh](https://wiki.jmonkeyengine.org/docs/3.4/core/scene/mesh.html)

!!! abstract "See [Mesh Javadoc](https://javadoc.ngengine.org/com/jme3/scene/Mesh.html)"

**Meshes** define the shape of a geometry through vertices, indices, normals, and texture coordinates.

!!! example
    ```java
    Mesh mesh = new Box(1, 1, 1); // Create a box mesh
    Geometry geometry = new Geometry("Box", mesh);
    geometry.setMaterial(material);
    rootNode.attachChild(geometry);
    ```
    
Meshes can be one of three things:

* **Shapes:** The simplest type of Meshes are NGE's default Shapes such as cubes and spheres. You can use several Shapes to build complex Geometries. Shapes are built-in and can be created without using the AssetManager.
* **3D Models:** 3D models and scenes are also made up of meshes, but are more complex than Shapes. You create Models and Scenes in external 3D Mesh Editors and export them as glTF. 
* **Custom Meshes:** Advanced users can create Custom Meshes programmatically.
