# 3-D Rotation


!!! info "From jME3 wiki"
    This is an excerpt from the jME3 wiki, check the original article for more details: [3D Rotation](https://wiki.jmonkeyengine.org/jme3/advanced/3d_rotation.html)
    

**3D rotation** is a mathematical operation where all vertices of your object are multiplied by four floating point numbers. This multiplication is called *concatenation*. The array of four numbers `{x, y, z, w}` is known as a [quaternion](https://javadoc.ngengine.org/com/jme3/math/Quaternion.html). Don’t worry—your 3D engine handles the hard part. All you need to know:

**The Quaternion** is an object that can "deep-freeze" and store a rotation which can then be applied to a 3D object.

!!! abstract "See [Quaternion Javadoc](https://javadoc.ngengine.org/com/jme3/math/Quaternion.html)"

## Using Quaternions for Rotation

To store a rotation in a quaternion, you must specify:

* The **angle** (as a multiple of π)
* The **axis** (as a vector—think *pitch*, *yaw*, *roll*)

**Example:**

```java
/* This quaternion stores a 180 degree rolling rotation */
Quaternion roll180 = new Quaternion();
roll180.fromAngleAxis(FastMath.PI, new Vector3f(0,0,1));
/* Apply the rotation */
thingamajig.setLocalRotation(roll180);
```

### Cheat-Sheet: Quaternion Axis Vectors

| Rotation around Axis? | Use this Axis Vector! | Examples                                       |
| --------------------- | --------------------- | ---------------------------------------------- |
| X axis                | (1, 0, 0)             | A plane pitches, nodding your head             |
| Y axis                | (0, 1, 0)             | A plane yaws, vehicle turns, shaking your head |
| Z axis                | (0, 0, 1)             | A plane rolls or banks, cocking your head      |

!!! note
    These are common examples—you can rotate around *any* axis expressed by a vector.

### Cheat-Sheet: Angle to Radian Conversion

| Angle       | Radian Value            | Examples                         |
| ----------- | ----------------------- | -------------------------------- |
| 45°         | `FastMath.PI / 4`       | Eighth of a circle               |
| 90°         | `FastMath.PI / 2`       | Quarter circle (3 o'clock)       |
| 180°        | `FastMath.PI`           | Half circle (6 o'clock)          |
| 270°        | `FastMath.PI * 3 / 2`   | Three-quarter circle (9 o'clock) |
| 360°        | `FastMath.PI * 2`       | Full circle (12 o'clock 😉)      |
| `g` degrees | `FastMath.PI * g / 180` | Any angle `g`                    |

!!! warning
    You must specify angles in [radians](http://en.wikipedia.org/wiki/Radian). Using degrees will lead to incorrect results.

### Steps to Specify a Rotation

1. Choose the axis vector.
2. Choose the radians value.
3. Create a quaternion: `fromAngleAxis(radians, vector)`
4. Apply it to your object: `setLocalRotation(...)`

Give your quaternion variables meaningful names like `roll90`, `pitch45`, `yaw180`.

More on [Using Quaternions](http://moddb.wikia.com/wiki/OpenGL:Tutorials:Using_Quaternions_to_represent_rotation)

---

## Code Sample

```java
/* Start with a horizontal object */
Cylinder cylinder = new Cylinder("post", 10, 10, 1, 10);
cylinder.setModelBound(new BoundingBox());

/* Create a 90-degree pitch rotation */
Quaternion pitch90 = new Quaternion();
pitch90.fromAngleAxis(FastMath.PI / 2, new Vector3f(1, 0, 0));

/* Apply the rotation */
cylinder.setLocalRotation(pitch90);

/* Update the model */
cylinder.updateModelBound();
cylinder.updateGeometry();
```

---

## Interpolating Rotations

You can interpolate between two rotations:

* `Quaternion.slerp()` stores a step between two rotations
  See [com.jme3.math.Quaternion](https://javadoc.ngengine.org/com/jme3/math/Quaternion.html)

---

## Adding Rotations

Concatenation (adding rotations):

```java
Quaternion myRotation = pitch90.mult(roll45); // pitch and roll
```

---

## Troubleshooting Rotations

If your object ends up in the wrong orientation:

1. **Order matters!** 3D transformations are *non-commutative*. Rotation before translation ≠ translation before rotation.
2. **Pivot points:** Want to rotate around a pivot *not* at the origin? Create a parent pivot node, attach your object to it, and rotate the parent.
3. **Radians vs degrees:** The engine expects *radians*. Convert degrees:
   `g° = FastMath.PI * g / 180`
4. **Do not modify constants!**


