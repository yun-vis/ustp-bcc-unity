---
# permalink: /about/
layout: single
title: "Rasterization"
classes: wide
header:
  image: /assets/images/teaser/teaser.png
  caption: "Image credit: [**Yun**](http://yun-vis.net)"
last_modified_at: 2023-04-18
---

# Rasterization

## DDA

GridMesh.cs
```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Serialization;

// Pre-setting before scripts
// Step1: Create a material, name White and choose shader "Sprites -> Default"

[RequireComponent(typeof(MeshFilter), typeof(MeshRenderer))]
public class GridMesh : MonoBehaviour
{
    public Mesh mesh;
    private Vector3[] _vertices;
    private Color[] _colors;
    private int[] _triangles;

    // Grid setting
    public float cellSize;
    public Vector3 gridoffset;
    public int gridSize;
    
    // Use this for initialization
    void Awake()
    {
        mesh = GetComponent<MeshFilter>().mesh;
        Vector3 gridPos = new Vector3(0, 0, 0);
        GetComponent<Transform>().position = gridPos;
        
        Debug.Log($"position = {GetComponent<Transform>().position}");
    }

    // Start is called before the first frame update
    void Start()
    {
        CreateProceduralGrid();
        UpdateMesh();
    }

    void CreateProceduralGrid()
    {
        // Set array sizes
        _vertices = new Vector3[4 * gridSize * gridSize];
        _colors = new Color[4 * gridSize * gridSize];
        _triangles = new int[6 * gridSize * gridSize];
        
        // Set tracker integers
        int v = 0;
        int t = 0;

        // Set vertex offset
        float vertexOffset = cellSize * 0.5f;

        for (int x = 0; x < gridSize; x++)
        {
            for (int z = 0; z < gridSize; z++)
            {
                Vector3 cellOffset = new Vector3(x * cellSize, 0, z * cellSize);
                
                _vertices[v + 0] = new Vector3(-vertexOffset, 0, -vertexOffset) + cellOffset + gridoffset;
                _vertices[v + 1] = new Vector3(-vertexOffset, 0, vertexOffset) + cellOffset + gridoffset;
                _vertices[v + 2] = new Vector3(vertexOffset, 0, -vertexOffset) + cellOffset + gridoffset;
                _vertices[v + 3] = new Vector3(vertexOffset, 0, vertexOffset) + cellOffset + gridoffset;
                
                _colors[v + 0] = _colors[v + 1] = _colors[v + 2] = _colors[v + 3] = new Color(1, 1, 1, 1);

                _triangles[t + 0] = v;
                _triangles[t + 1] = _triangles[t + 4] = v + 1;
                _triangles[t + 2] = _triangles[t + 3] = v + 2;
                _triangles[t + 5] = v + 3;

                v += 4;
                t += 6;
            }
        }
    }

    void UpdateMesh()
    {
        // Clear all vertex data and all triangle indices
        mesh.Clear();
        // Assign our vertices and triangles to mesh object
        mesh.vertices = _vertices;
        mesh.colors = _colors;
        mesh.triangles = _triangles;

        mesh.RecalculateNormals();
    }

    // Update is called once per frame
    void Update()
    {
    }
}
```

DDA.cs
```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

// Pre-setting before scripts
// Step1: Create an empty game object, name Source and add material blue
// Step2: Create an empty game object, name Target and add material red
// Step3: Create an empty game object, name Grid and attach GridMesh script and add material white
// Step4: Create an empty game object, name DDA and attach DDA script

public class DDA : MonoBehaviour
{
    private GameObject _source;
    private GameObject _target;
    private GameObject _gridMesh;
    private Mesh _mesh;
    private int _gridSize;
    private float _cellSize;

    // Use this for initialization
    void Awake()
    {
        // Get the GameObject information
        _source = GameObject.Find("Source");
        _target = GameObject.Find("Target");
        _gridMesh = GameObject.Find("Grid");
        _mesh = _gridMesh.GetComponent<MeshFilter>().mesh;
        _gridSize = _gridMesh.GetComponent<GridMesh>().gridSize;
        _cellSize = _gridMesh.GetComponent<GridMesh>().cellSize;

        // Reset grid position
        Vector3 gridPosition = _gridMesh.transform.position = new Vector3(0, 0, 0);

        // Reset source and target position
        Vector3 sourcePos = new Vector3(
            gridPosition.x,
            gridPosition.y,
            gridPosition.z
        );
        Vector3 targetPos = new Vector3(
            (_gridSize - 1) * _cellSize,
            gridPosition.y,
            (_gridSize - 1) * _cellSize
        );

        // Initialize positions
        _source.transform.position = sourcePos;
        _target.transform.position = targetPos;
    }

    // Start is called before the first frame update
    void Start()
    {
    }

    // Update is called once per frame
    void Update()
    {
        Vector3 gridPosition = _gridMesh.transform.position;

        // Update positions
        Vector3 sourcePos = _source.transform.position;
        sourcePos.y = gridPosition.y;
        Vector3 targetPos = _target.transform.position;
        targetPos.y = gridPosition.y;

        // Debug.Log($"sourcePos = {sourcePos }");
        _source.transform.position = sourcePos;
        _target.transform.position = targetPos;

        ComputeDDA();
    }

    void ComputeDDA()
    {
        Vector3 sourcePos = _source.transform.position;
        Vector3 targetPos = _target.transform.position;

        // Create new colors array where the colors will be created.
        Color[] colors = _mesh.colors;
        // Reset colors of the grid mesh to white
        for (int i = 0; i < colors.Length; i++)
        {
            colors[i].r = colors[i].g = colors[i].b = colors[i].a = 1;
        }

        float dx = (targetPos.x - sourcePos.x);
        float dz = (targetPos.z - sourcePos.z);
        //float dx = (targetPos.x - sourcePos.x) / _cellSize;
        //float dz = (targetPos.z - sourcePos.z) / _cellSize;
        float m = dz / dx;

        float x = sourcePos.x;
        float z = sourcePos.z;
        //float x = sourcePos.x / _cellSize;
        //float z = sourcePos.z / _cellSize;
        DrawPixel(colors, (int)Math.Round(x), (int)Math.Round(z));
        // m <= 1
        if (m <= 1)
        {
            for (int i = 0; i < dx; i++)
            {
                x += 1;
                z += m;
                DrawPixel(colors, (int)Math.Round(x), (int)Math.Round(z));
            }
        }
        // m > 1
        else
        {
            m = dx / dz;
            for (int i = 0; i < dz; i++)
            {
                x += m;
                z += 1;
                DrawPixel(colors, (int)Math.Round(x), (int)Math.Round(z));
            }
        }

        // Update the grid colors
        _mesh.colors = colors;
    }

    void DrawPixel(Color[] colors, int x, int z)
    {
        // Debug.Log($"(x,z) = ({x},{z})");
        if (x >= _gridSize || z >= _gridSize) return;

        // Debug.Log($"vertices.Length = {vertices.Length}");
        // Assign the array of colors to the Mesh.
        int v = 4 * (x * _gridSize + z);

        colors[v] = new Color(0, 0, 0, 1);
        colors[v + 1] = new Color(0, 0, 0, 1);
        colors[v + 2] = new Color(0, 0, 0, 1);
        colors[v + 3] = new Color(0, 0, 0, 1);
    }
}
```

## Bresenham's Line Algorithm

Bresenham.cs
```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

// Pre-setting before scripts
// Step1: Create an empty game object, name Source and add material blue
// Step2: Create an empty game object, name Target and add material red
// Step3: Create an empty game object, name Grid and attach GridMesh script and add material white
// Step4: Create an empty game object, name DDA and attach Bresenham script

public class Bresenham : MonoBehaviour
{
    private GameObject _source;
    private GameObject _target;
    private GameObject _gridMesh;
    private Mesh _mesh;
    private int _gridSize;
    private float _cellSize;
    
    // Use this for initialization
    void Awake()
    {
        // Get the GameObject information
        _source = GameObject.Find("Source");
        _target = GameObject.Find("Target");
        _gridMesh = GameObject.Find("Grid");
        _mesh = _gridMesh.GetComponent<MeshFilter>().mesh;
        _gridSize = _gridMesh.GetComponent<GridMesh>().gridSize;
        _cellSize = _gridMesh.GetComponent<GridMesh>().cellSize;

        // Reset grid position
        Vector3 gridPosition = _gridMesh.transform.position = new Vector3(0, 0, 0);

        // Reset source and target position
        Vector3 sourcePos = new Vector3(
            gridPosition.x,
            gridPosition.y,
            gridPosition.z
        );
        Vector3 targetPos = new Vector3(
            (_gridSize - 1) * _cellSize,
            gridPosition.y,
            (_gridSize - 1) * _cellSize
        );

        // Initialize positions
        _source.transform.position = sourcePos;
        _target.transform.position = targetPos;
    }

    // Start is called before the first frame update
    void Start()
    {
    }

    // Update is called once per frame
    void Update()
    {
        Vector3 gridPosition = _gridMesh.transform.position;

        // Update positions
        Vector3 sourcePos = _source.transform.position;
        sourcePos.y = gridPosition.y;
        Vector3 targetPos = _target.transform.position;
        targetPos.y = gridPosition.y;

        // Debug.Log($"sourcePos = {sourcePos }");
        _source.transform.position = sourcePos;
        _target.transform.position = targetPos;

        ComputeBresenham();
    }

    void ComputeBresenham()
    {
        Vector3 sourcePos = _source.transform.position;
        Vector3 targetPos = _target.transform.position;

        // Create new colors array where the colors will be created.
        Color[] colors = _mesh.colors;
        // Reset colors of the grid mesh to white
        for (int i = 0; i < colors.Length; i++)
        {
            colors[i].r = colors[i].g = colors[i].b = colors[i].a = 1;
        }

        // 1. store left line endpoint in (x0,y0)
        int xk = (int)(sourcePos.x / _cellSize);
        int zk = (int)(sourcePos.z / _cellSize);
        int dx = (int)((targetPos.x - sourcePos.x) / _cellSize);
        int dz = (int)((targetPos.z - sourcePos.z) / _cellSize);

        // 2. draw pixel  (x0,y0) 
        DrawPixel(colors, xk, zk);

        // m <= 1
        if (dz <= dx)
        {
            // 3. calculate constants  Dx, Dy, 2Dy, 2Dy - 2Dx, and obtain  p0 = 2Dy - Dx
            int twodz_twodx = 2 * dz - 2 * dx;
            int twodz = 2 * dz;
            int pk = 2 * dz - dx;

            // 4. at each xk along the line, perform test:
            // if pk<0 
            // then draw pixel (xk+1,yk);  pk+1 = pk+ 2Dy
            // else draw pixel (xk+1,yk+1);  pk+1= pk+ 2Dy - 2Dx
            // 5. perform “step 4”  (Dx - 1) times.
            for (int i = 0; i < dx; i++)
            {
                if (pk < 0)
                {
                    DrawPixel(colors, xk + 1, zk);
                    pk = pk + twodz;
                }
                else
                {
                    DrawPixel(colors, xk + 1, zk + 1);
                    zk++;
                    pk = pk + twodz_twodx;
                }

                xk++;
            }
        }
        // m <= 1
        else
        {
            // 3. calculate constants  Dx, Dy, 2Dy, 2Dy - 2Dx, and obtain  p0 = 2Dy - Dx
            int twodx_twodz = 2 * dx - 2 * dz;
            int twodx = 2 * dx;
            int pk = 2 * dx - dz;

            // 4. at each xk along the line, perform test:
            // if pk<0 
            // then draw pixel (xk+1,yk);  pk+1 = pk+ 2Dy
            // else draw pixel (xk+1,yk+1);  pk+1= pk+ 2Dy - 2Dx
            // 5. perform “step 4”  (Dx - 1) times.
            for (int i = 0; i < dz; i++)
            {
                if (pk < 0)
                {
                    DrawPixel(colors, xk, zk + 1);
                    pk = pk + twodx;
                }
                else
                {
                    DrawPixel(colors, xk + 1, zk + 1);
                    xk++;
                    pk = pk + twodx_twodz;
                }

                zk++;
            }
        }

        // Update the grid colors
        _mesh.colors = colors;
    }

    void DrawPixel(Color[] colors, int x, int z)
    {
        // Debug.Log($"(x,z) = ({x},{z})");
        if (x >= _gridSize || z >= _gridSize) return;

        // Debug.Log($"vertices.Length = {vertices.Length}");
        // Assign the array of colors to the Mesh.
        int v = 4 * (x * _gridSize + z);

        colors[v] = new Color(0, 0, 0, 1);
        colors[v + 1] = new Color(0, 0, 0, 1);
        colors[v + 2] = new Color(0, 0, 0, 1);
        colors[v + 3] = new Color(0, 0, 0, 1);
    }
}
```

---
# External Resources

## [SerializeField] annotation [Doc](https://mobidictum.com/game-industry/unity-serializefield/)

