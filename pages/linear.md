---
# permalink: /about/
layout: single
title: "Linear Primitives"
classes: wide
header:
  image: /assets/images/teaser/teaser.png
  caption: "Image credit: [**Yun**](http://yun-vis.net)"
last_modified_at: 2025-05-10
---

# Raycasting

Raycasting is a method used in computer graphics to render 3D scenes by tracing light rays from a viewpoint into the scene. It's essentially the opposite of ray tracing, which traces rays from light sources to the viewer. In simpler terms, raycasting simulates how light will hit the eye or camera in a virtual environment. 

## Raycasting in Unity

Raycasting is a technique for detecting collisions in 3D space by simulating a "laser beam" from a point in space. It's used to determine what objects, if any, a line segment intersects with. This information can be used for various game mechanics, like shooting, selecting objects, or checking for obstacles. 

* Pre-setting before scripts
  * Step1: Create three Cube GameObeject called Wall1-Wall3. One can change the sizes of the walls to make it reasonable.
    * Wall1: Position(1.5,0,-1.5), Scale(3,1,1)
    * Wall2: Position(0,0,1), Scale(1,1,3), set layer in the inspector to World
    * Wall3: Position(1.5,0,3), Scale(2,1,1)
  * Step2: Create another Cube GameObject "RayOrigin" and attach the script Raycasting
    * RayOrigin: Position(5,0,0)
  * Step3: Create another Cube GameObject "RayReflection" and attach the script RaycastingReflection
    * RayOrigin: Position(5,0,0)
  * Step4: Reflection set to 3 and MaxLength set to 100
  * Step5: Add "Mirror" tag to the Wall 1-3
  * Step6: Enable LineRenderer.CornerVertices to make the line looks smoother. i.e., 10.
  * Step7: Enable LineRenderer.EndCapVertices to make the line looks smoother. i.e., 10.
  * Step8: Change the width or add material...
    
### Unity Layer
Layers are a tool that allows you to separate GameObjects in your scenes. You can use layers through the UI and with scripts to edit how GameObjects within your scene interact with each other.

Raycasting.cs
```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class Raycasting : MonoBehaviour
{
    private Ray _ray;
    // Container for hit data
    private RaycastHit _hitData;
    public LayerMask layers;

    // Initialization, usually instantiation of objects
    // and setting up references to other objects.
    private void Awake()
    {
    }

    // Start is called before the first frame update
    void Start()
    {
        // Creates a Ray from this object, moving forward
        // Vector3.left for the X-Axis, Vector3.up for the Y-Axis and Vector3.forward for the Z-Axis.
        // _ray = new Ray(transform.position, transform.forward);

        // Creates a Ray from the center of the viewport
        // _ray = Camera.main.ViewportPointToRay(new Vector3 (0.5f, 0.5f, 0));

        // Creates a Ray from the mouse position
        // _ray = Camera.main.ScreenPointToRay(Input.mousePosition);

        layers = LayerMask.GetMask("Default");
        // layers = LayerMask.GetMask("World") | LayerMask.GetMask("Water");
        // layers = 1<<9;
        // Debug.Log("test!");
    }

    // Update is called once per frame
    void Update()
    {
        FireRay();
    }

    void FireRay()
    {
        // Set ray origin and direction
        _ray = new Ray(transform.position, transform.forward);
        // Visualize the ray in debug
        Debug.DrawRay(_ray.origin, _ray.direction * 10);

        // One of the most common ways to use Raycast is using the Physics Class, which returns a boolean true or false, depending on if the Ray hit anything.
        // The out keyword in C# is used to return extra information from a function.
        // ref requires the variable to be initialized before being passed to the method, whereas out does not, and the method is responsible for initializing it before returning.
        if (Physics.Raycast(_ray, out _hitData, 10, layers))
        {
            Vector3 hitPosition = _hitData.point;
            float hitDistance = _hitData.distance;
            // Reads the Collider name
            string name = _hitData.collider.name;
            // Gets a Game Object reference from its Transform
            GameObject hitObject = _hitData.transform.gameObject;

            // Debug.Log("hitted object name = " + name);
            Debug.Log("hitted object name = " + hitObject.name);
        }
    }
}
```

## Raycasting Reflection

RaycastReflection.cs
```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

[RequireComponent(typeof(LineRenderer))]
public class RaycastReflection : MonoBehaviour
{
    [SerializeField]
    private int _reflections;
    [SerializeField]
    private float _maxLength;

    private LineRenderer _lineRenderer;
    private Ray _ray;
    private RaycastHit _hitData;
    // private Vector3 _direction;

    void Awake()
    {
        _lineRenderer = GetComponent<LineRenderer>();
    }

    // Start is called before the first frame update
    void Start()
    {
    }

    // Update is called once per frame
    void Update()
    {
        // Set ray origin and direction
        _ray = new Ray(transform.position, transform.forward);

        // initialization of the line renderer
        // positionCount returns the number of vertices in the line.
        _lineRenderer.positionCount = 1;
        // Set the position of a vertex in the line.
        _lineRenderer.SetPosition(0, transform.position);
        float remainingLength = _maxLength;

        for (int i = 0; i < _reflections; i++)
        {
            // retrun true if the ray hits something
            if(Physics.Raycast(_ray, out _hitData, remainingLength))
            {
                _lineRenderer.positionCount += 1;
                _lineRenderer.SetPosition(_lineRenderer.positionCount-1, _hitData.point);
                remainingLength -= Vector3.Distance(_ray.origin, _hitData.point);

                if (_hitData.collider.tag == "Mirror"){
                    // _hitData.normal stores the normal of the surface the ray hit. 
                    _ray = new Ray(_hitData.point, Vector3.Reflect(_ray.direction, _hitData.normal));
                }
            }
            else
            {
                // Calculate the remaining length of the ray
                _lineRenderer.positionCount += 1;
                _lineRenderer.SetPosition(_lineRenderer.positionCount - 1, _ray.origin + _ray.direction * remainingLength);
            }
        }        
        // Debug.Log("positionCount: " + _lineRenderer.positionCount);
    }
}
```
  
# Curve

## Bézier Curve in Unity

* Pre-setting before scripts
  * Step1: Create an empty game object, name Curve.
  * Step2: Create 4 empty game objects, name ControlPoint1-4 and select icon for them (under inspector text).
  * Step3: Drag the curve script to the Curve game object and set control points to 4
  * Step4: Drag control points to the corresponding control point elements

Curve2D.cs
```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

// Pre-setting before scripts
// Step1: Create an empty game object, name Curve.
// Step2: Create 4 empty game objects, name ControlPoint1-4 and select icon for them (under inspector text).
// Step3: Drag the curve script to the Curve game object and set control points to 4
// Step4: Drag control points to the corresponding control point elements

public class Curve2D : MonoBehaviour
{
    [SerializeField]
    private Transform[] controlPoints;
    
    // Start is called before the first frame update
    void Start()
    {
        
    }

    // Update is called once per frame
    void Update()
    {
        
    }
    
    private void OnDrawGizmos()
    {
        // Draw the interpolated points using spheres
        for(float t = 0; t <= 1; t += 0.05f)
        {
            Vector2 gizmosPosition = Mathf.Pow(1 - t, 3) * controlPoints[0].position + 3 * Mathf.Pow(1 - t, 2) * t * controlPoints[1].position + 3 * (1 - t) * Mathf.Pow(t, 2) * controlPoints[2].position + Mathf.Pow(t, 3) * controlPoints[3].position;

            Gizmos.DrawSphere(gizmosPosition, 0.25f);
        }

        // Draw the lines between the control points
        Gizmos.DrawLine(controlPoints[0].position, controlPoints[1].position);
        Gizmos.DrawLine(controlPoints[2].position, controlPoints[3].position);
        // Gizmos.DrawLine(new Vector2(controlPoints[0].position.x, controlPoints[0].position.y), new Vector2(controlPoints[1].position.x, controlPoints[1].position.y));
        // Gizmos.DrawLine(new Vector2(controlPoints[2].position.x, controlPoints[2].position.y), new Vector2(controlPoints[3].position.x, controlPoints[3].position.y));
    }
}
```

Curve3D.cs
```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;


public class Curve3D : MonoBehaviour
{
    [SerializeField]
    private Transform[] controlPoints;

    // Start is called before the first frame update
    void Start()
    {

    }

    // Update is called once per frame
    void Update()
    {

    }

    private void OnDrawGizmos()
    {
        // Draw the interpolated points using spheres
        for (float t = 0; t <= 1; t += 0.05f)
        {
             Vector3 gizmosPosition = Mathf.Pow(1 - t, 3) * controlPoints[0].position + 3 * Mathf.Pow(1 - t, 2) * t * controlPoints[1].position + 3 * (1 - t) * Mathf.Pow(t, 2) * controlPoints[2].position + Mathf.Pow(t, 3) * controlPoints[3].position;

            Gizmos.DrawSphere(gizmosPosition, 0.25f);
        }

        // Draw the lines between the control points
        Gizmos.DrawLine(controlPoints[0].position, controlPoints[1].position);
        Gizmos.DrawLine(controlPoints[2].position, controlPoints[3].position);

        // Gizmos.DrawLine(new Vector3(controlPoints[0].position.x, controlPoints[0].position.y, controlPoints[0].position.z), new Vector3(controlPoints[1].position.x, controlPoints[1].position.y, controlPoints[1].position.z));
        // Gizmos.DrawLine(new Vector3(controlPoints[2].position.x, controlPoints[2].position.y, controlPoints[2].position.z), new Vector3(controlPoints[3].position.x, controlPoints[3].position.y, controlPoints[3].position.z));
    }
}
```

* Pre-setting before scripts
  * Step1: Create a cube game object called Follower and attach the script to it.
  * Step2: Change the Route size in the inspector to 1 and drag the Curve object there.
  * Step3: Create the 2nd Curve object and change the Route size to 2 in the inspector.

CurveFollow.cs
```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class CurveFollow : MonoBehaviour
{
    [SerializeField]
    Transform[] routes;

    private int _routeToFollow;
    private float _tParam;
    private float _speedModifier;
    private bool _coroutineAllowed;

    // Start is called before the first frame update
    void Start()
    {
        _routeToFollow = 0;
        _tParam = 0f;
        _speedModifier = 0.3f;
        _coroutineAllowed = true;
    }

    // Update is called once per frame
    void Update()
    {
        if (_coroutineAllowed)
        {
            StartCoroutine(FollowTheRoute(_routeToFollow));
        }
    }

    // This function will be executed across the frames    
    private IEnumerator FollowTheRoute(int routeNum)
    {
        _coroutineAllowed = false;

        Vector3 p0 = routes[routeNum].GetChild(0).position;
        Vector3 p1 = routes[routeNum].GetChild(1).position;
        Vector3 p2 = routes[routeNum].GetChild(2).position;
        Vector3 p3 = routes[routeNum].GetChild(3).position;

        while (_tParam < 1)
        {
            _tParam += Time.deltaTime * _speedModifier;

            transform.position = Mathf.Pow(1 - _tParam, 3) * p0 + 3 * Mathf.Pow(1 - _tParam, 2) * _tParam * p1 + 3 * (1 - _tParam) * Mathf.Pow(_tParam, 2) * p2 + Mathf.Pow(_tParam, 3) * p3;

            // Can be used to wait until the end of the frame to have a screenshot
            // yield return new WaitForEndOfFrame(); 
            // yield return new WaitForSeconds(0.03f);
            // Suspend the coroutine until the next frame
            yield return null;
        }

        // Reset the tParam to 0 and move to the next route
        _tParam = 0.0f;
        _routeToFollow += 1;
        _routeToFollow %= routes.Length;
        _coroutineAllowed = true;
    }
}
```

---
# External Resources

