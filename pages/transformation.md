---
# permalink: /about/
layout: single
title: "Transformation"
classes: wide
header:
  image: /assets/images/teaser/teaser.png
  caption: "Image credit: [**Yun**](http://yun-vis.net)"
last_modified_at: 2026-05-11
---

# Prerequisites

* No need to use scripts from the previous lecture.

# GameObjects

Every object in your game is a GameObject, from characters and collectible items to lights, cameras and special effects. For better understanding, tackle **Move/Transform Tool** in the Scene view toolbar.
 
* Position: Position of the Transform in the x, y, and z coordinates.
* Rotation: Rotation of the Transform around the x-axis, y-axis, and z-axis, measured in degrees.
* Scale: Scale of the Transform along the x-axis, y-axis, and z-axis. The value “1” is the original size (the size at which you imported the GameObject).
* Enable Constrained Proportions: Force the scale to maintain its current proportions, so that changing one axis changes the other two axes. Disabled by default. One can find the button next to the [Scale](https://docs.unity3d.com/2023.2/Documentation/Manual/class-ScaleConstraint.html) in the inspector.

# Gizmo

In Unity, gizmos are visual aids drawn in the Scene view to help with debugging and visualization during development, not during runtime. 

* For position: Click the Pivot/Center button on the left to toggle between Pivot and Center.
  * Pivot positions the Gizmo at the actual pivot point of the parent GameObject, as defined by the Transform component.
  * Center positions the Gizmo at a center position based on the selected GameObjects.
* For rotation: Click the Local/Global button on the right to toggle between Local and Global.
  * Local keeps the Gizmo’s rotation relative to the GameObject’s.
  * Global clamps the Gizmo to world space orientation.
* Between Center and Pivot. Select the Translate tool.
  * Step1: Create 2 cubes (e.g., Cube1, Cube2) from the 3D obecjt and observe the similarity
  * Step2: Child Cube2 object to another Cube1. Now one see the difference
* Between Global and Local. Select the Rotate tool.
  * Step1: Select one cube object.
  * Step2: Choose the Local mode and rotate the cube. Rotate it to non-trival angles.
  * Step3: Choose the Global mode, observe the Gizmo, and rotate the cube again.

# Rotation

## Euler Angle vs. Quaternion

* Pre-setting before scripts
  * Step1: Create a GameObeject MyCube thats is composed of multiple cubes (e.g., 3 cubes). Press "v" to enable vertex snapping during translation.
  <!-- * Step2: Set up Pivot vs. Center (calculated by BoundingBox) of the another two empty GameObject
    * What is a Bounding Box? A bounding box (Axis-Aligned Bounding Box and Oriented Bounding Box) is an automatically-created invisible box that defines the rough size of an entity. -->

* Global vs local rotation
  * Step1: Create a script Assets/Scripts/Transformation/Rotation.cs and attach it to MyCubes
  * Step2: Create an empty game object, name Containter and drag MyCubes to be the child

* Pivot point
  * Step1: Add a method RotateAroundPoint() in the script

In Assets/Scripts/Transformation/Rotation.cs
```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Rotation : MonoBehaviour
{
    // For animation
    private float _degreesPerSecond = 20.0f;
    private Transform _parentTransform;

    // Initialization, usually instantiation of objects
    // and setting up references to other objects.
    private void Awake()
    {
        _parentTransform = transform.parent;
    }

    // Start is called before the first frame update
    void Start()
    {
        // Change object position
        transform.position = new Vector3(0, 0, 0);

        // Change object rotation in Euler angles in Global space
        // The rotation as Euler angles in degrees.
        transform.eulerAngles = new Vector3(0, 45, 0);
        // transform.eulerAngles = new Vector3(0, transform.eulerAngles.y + 45, 0);
        // transform.eulerAngles = new Vector3(transform.eulerAngles.x, transform.eulerAngles.y + 45, transform.eulerAngles.z);

        // There are in principle 2 types of rotations in CG. 
        // One is using Euler angle, and the other is using Quaternion. 
        // Behind the scenes, Unity calculates rotation using Quaternions,
        // meaning that the rotation property of an object is not actually a Vector 3,
        // it’s a Quaternion.

        // The rotation value you see in the Inspector is the real Quaternion rotation
        // converted to a Vector 3 value, an Euler Angle,
        // which is rotation expressed as an XYZ value for pitch, yaw and roll.

        // Copies another object's rotation
        // Quaternion objectRotation = _otherObject.transform.rotation;
        // transform.rotation = objectRotation;

        // Instead of working in Quaternions, you can convert a Vector 3 Euler Angle rotation into a Quaternion.
        // A Quaternion that stores the rotation of the Transform in Global space.
        // equal to transform.eulerAngles =  new Vector3(0, 45, 0);
        transform.rotation = Quaternion.Euler(new Vector3(0, 45, 0));
        // Quaternion.operator *
        // https://docs.unity3d.com/ScriptReference/Quaternion-operator_multiply.html
        // transform.rotation = Quaternion.Euler(new Vector3(0, 45, 0)) * Quaternion.Euler(new Vector3(0, 45, 0));

        // Generally speaking, Unity uses Quaternions because they’re efficient and avoid
        // some of the problems of using Euler Angles, such as gimbal lock, which is the
        // loss of a degree of movement when two axes are aligned the same way.

        // Local rotation vs Global rotation
        // By default, when setting rotation directly, you’re setting the object’s Global rotation
        // Setting the Global rotation of a child object will rotate it into that position absolutely.

        // Local position/rotation/scale, is that transformation relative to its parent.
        // Global is the combination of a GameObject's local transformation with all of
        // its parents, to the root of the scene.

        // Global Rotation
        // With/without roatating the parent object Container -30 degrees around the X-Axis
        // transform.eulerAngles = new Vector3(0, 45, 0);
        // transform.eulerAngles = new Vector3(transform.eulerAngles.x, 45, transform.eulerAngles.z);
        // same as above
        transform.rotation = Quaternion.Euler(new Vector3(0, 45, 0));
        // transform.rotation = Quaternion.Euler(new Vector3(transform.eulerAngles.x, 45, transform.eulerAngles.z));

        // Local Rotation
        // With/without roatating the parent object Container -30 degrees around the X-Axis
        // Debug.Log(transform.localRotation.eulerAngles.y);
        // transform.localEulerAngles = new Vector3(0, 0, transform.localRotation.eulerAngles.y-30);
        transform.localEulerAngles = new Vector3(0, 45, 0);
        // same as above
        // transform.localRotation = Quaternion.Euler(new Vector3(0, 45, 0));

        // Creating a parent object is an easy way to move the pivot point of an object.
        // Unchild Container and MyCubes
        // case 1: 
        transform.parent.rotation = Quaternion.Euler(new Vector3(0, 15, 0));
        transform.localRotation = Quaternion.Euler(new Vector3(0, 30, 0));
        // The game object will end up with 45 degrees (rotated along y-axis.) This will equal to

        // case 2: 
        transform.rotation = Quaternion.Euler(new Vector3(0, 45, 0));

        // case 3: 
        transform.parent.rotation = Quaternion.Euler(new Vector3(0, 15, 0));
        transform.rotation = Quaternion.Euler(new Vector3(0, 45, 0));
        // because in case 3, the angle in Global space overwrite the angle in Local space.
    }

    // Update is called once per frame
    void Update()
    {
        // RotateAroundPivot();
        RotateAroundPoint();
    }

    void RotateAroundPivot()
    {
        // Time.deltaTime (the duration of the last frame) to make sure that the rotation is smooth and consistent.
        // This is to make sure that, even if the framerate changes, the amount of movement over time remains the same.
        // In this case, it changes the amount of rotation from 20 degrees per frame to 20 degrees per second.
        // transform.Rotate(new Vector3(0, degreesPerSecond, 0) * Time.deltaTime, Space.Self);
        // Space.Self means that the rotation is relative to the object’s local space.
        transform.Rotate(new Vector3(0, _degreesPerSecond, 0) * Time.deltaTime, Space.Self);
    }

    void RotateAroundPoint()
    {
        // Vector3 pivotPoint = new Vector3(0, 0, 0);
        Vector3 pivotPoint = new Vector3(-1.5f, 0, 0.5f);
        // Vector3 pivotPoint = _parentTransform.position;
        Debug.Log("Pivot Point: " + pivotPoint);

        // Rotates around the pivot point and the Y-Axis by 90 degrees
        // Vector3.left for the X-Axis, Vector3.up for the Y-Axis and Vector3.forward for the Z-Axis.
        transform.RotateAround(pivotPoint, Vector3.up, _degreesPerSecond * Time.deltaTime);
    }
}
```

Potential errors: If the result does not look quite right, check if the parent postions are all at (0,0,0).

* RotateAround
  * Step1: Create an empty game object, name SolarSystem.
  * Step2: Create three sphere game objects, name Sun (s=1), Earth (s=0.5, z=2.0), and Moon (s=0.3, z=1.5). Drag game objects to be nested in hierarchy of SolarSystem. Moon appears to the the child of Earth, and Earth is the child of the Sun. Attach the script to Earth and later Moon.
  
In Assets/Scripts/Transformation/SolarSystem.cs
```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;


public class SolarSystem : MonoBehaviour
{
    // For animation
    private float _degreesPerSecond = 20.0f;
    private Transform _parentTransform;
    private Vector3 _direction;
    private float _angle;
    private float _radius;

    // Initialization
    void Awake()
    {
        // transform.position = new Vector3(0, 0, 0);
        _parentTransform = transform.parent;

        // Unit vector
        _direction = (transform.position - _parentTransform.position).normalized;
        // Distance between the parent and the object
        _radius = Vector3.Distance(_parentTransform.position, transform.position);
    }

    // Start is called before the first frame update
    void Start()
    {
    }

    // Update is called once per frame
    void Update()
    {
        //RotateOrbit();
        RotateOrbitSlant();
    }

    internal void RotateOrbit()
    {
        Debug.Log(_parentTransform.position);
        // Revolution
        transform.RotateAround(_parentTransform.position, Vector3.up, _degreesPerSecond * Time.deltaTime);
        // Rotation
        // Difference with RotateOrbitSlant(): the parent angle will be added to children angle
        // transform.Rotate( Vector3.up, 0.5f);
        // transform.Rotate( Vector3.up, 0.0f);
    }

    internal void RotateOrbitSlant()
    {
        _angle += _degreesPerSecond * Time.deltaTime;
        // _angle %= 360;
        // Debug.Log("angle = " + angle);

        // Rotate a vector by multiplying it by a Quaternion
        // Vector3.forward Z-Axis unit vector
        Vector3 orbit = Vector3.forward * _radius;
        // orbit = Quaternion.Euler(0, angle, 0) * orbit;
        orbit = Quaternion.LookRotation(_direction) * Quaternion.Euler(0, _angle%360, 0) * orbit;
        // Revolution
        transform.position = _parentTransform.position + orbit;
        // Rotation
        // transform.Rotate( Vector3.up, 0.5f);
    }
}
```

---
# External Resources

## Quaternion

- Understanding Quaternions [Doc](https://www.allaboutcircuits.com/technical-articles/dont-get-lost-in-deep-space-understanding-quaternions/)

- Quaternions and Rotations [Doc](https://graphics.stanford.edu/courses/cs348a-17-winter/Papers/quaternion.pdf)