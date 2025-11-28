---
# permalink: /about/
layout: single
title: "Deformation"
classes: wide
header:
  image: /assets/images/teaser/teaser.png
  caption: "Image credit: [**Yun**](http://yun-vis.net)"
last_modified_at: 2023-05-03
---

# Free-form Deformation

MeshGenerator.cs
```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

namespace TwoDMeshGenerator
{
    public class MeshVertexProperty
    {
        public int[] cornerPoints = new int[] { 0, 0, 0, 0 };

        public MeshVertexProperty(int lb, int rb, int rt, int lt)
        {
            cornerPoints[0] = lb;
            cornerPoints[1] = rb;
            cornerPoints[2] = rt;
            cornerPoints[3] = lt;
        }
    }

    public class MeshGenerator
    {
        public Vector3[] fixedVertices;
        public Vector3[] vertices;
        public int[] triangles;
        public MeshVertexProperty[] properties;

        void InitializeMeshData()
        {
            // create an array of viertices
            fixedVertices = new Vector3[]
            {
                new Vector3(-6, 0.01f, -2),
                new Vector3(-2, 0.01f, 1),
                new Vector3(0, 0.01f, -2),
                new Vector3(1, 0.01f, 2),
                new Vector3(3, 0.01f, -1),
                new Vector3(6, 0.01f, 1)
            };

            vertices = new Vector3[]
            {
                new Vector3(-6, 0.01f, -2),
                new Vector3(-2, 0.01f, 1),
                new Vector3(0, 0.01f, -2),
                new Vector3(1, 0.01f, 2),
                new Vector3(3, 0.01f, -1),
                new Vector3(6, 0.01f, 1)
            };

            // create an array of indices
            triangles = new[]
            {
                0, 1, 2,
                1, 3, 2,
                2, 3, 4,
                4, 3, 5
            };

            // create mesh vertex property
            properties = new[]
            {
                new MeshVertexProperty(0, 0, 0, 0),
                new MeshVertexProperty(0, 0, 0, 0),
                new MeshVertexProperty(0, 0, 0, 0),
                new MeshVertexProperty(0, 0, 0, 0),
                new MeshVertexProperty(0, 0, 0, 0),
                new MeshVertexProperty(0, 0, 0, 0)
            };
        }

        public void CreateMesh()
        {
            // initialize the mesh
            InitializeMeshData();
        }
    }
}
```

FFD.cs
```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using System.Linq;
using UnityEngine;

// third-party namespace
using QuikGraph;
// self-defined namespace
using Property;
using TwoDMeshGenerator;

[RequireComponent(typeof(MeshFilter), typeof(MeshRenderer))]
public class FFD : MonoBehaviour
{
    // variables for FFD control points
    public int xPointNumber = 5;
    public int zPointNumber = 3;

    private AdjacencyGraph<VertexProperty, TaggedEdge<VertexProperty, EdgeProperty>> graph;
    private List<GameObject> _controlPoints;
    private string _controlPointName = "ControlPoint";
    private const float _shift = 4.0f;

    // mesh for deformation
    private Mesh _mesh;
    private MeshGenerator _gen;

    // Start is called before the first frame update
    void Start()
    {
        initializeGridGraph();
        initializeControlPoints();
        initialize2DMesh();
    }

    // Update is called once per frame
    void Update()
    {
        if (graph != null && _mesh != null)
        {
            // update control points
            updateControlPointPosition();
            // update mesh boundary presentation
            updateMeshPosition();
        }
    }

    void initializeControlPoints()
    {
        _controlPoints = new List<GameObject>();
        for (int i = 0; i < xPointNumber * zPointNumber; i++)
        {
            // create sphere game objects
            GameObject obj = GameObject.CreatePrimitive(PrimitiveType.Sphere);
            obj.name = _controlPointName + "[" + i.ToString() + "]";
            obj.transform.parent = transform;
            Vector3 position = graph.Vertices.ElementAt(i).position;
            obj.transform.position = new Vector3(position.x, position.y, position.z);
            obj.transform.localScale = new Vector3(0.5f, 0.5f, 0.5f);
            _controlPoints.Add(obj);
        }
    }

    void initialize2DMesh()
    {
        _gen = new MeshGenerator();
        _mesh = GetComponent<MeshFilter>().mesh;
        _gen.CreateMesh();
        _mesh.vertices = _gen.vertices;
        _mesh.triangles = _gen.triangles;

        // initialize the corner points of each mesh vertex
        for (int i = 0; i < _mesh.vertices.Length; i++)
        {
            Vector3 position = _mesh.vertices[i];
            Vector3 diff = (position - graph.Vertices.ElementAt(0).position);

            // Debug.Log("diff.x/shift = " + (diff.x/shift));
            // Debug.Log("diff.z/shift = " + (diff.z/shift));
            // store the corner point id of each vertex in a mesh
            int id = (int)(diff.z / _shift) + zPointNumber * (int)(diff.x / _shift);
            _gen.properties[i].cornerPoints[0] = id; // lb
            _gen.properties[i].cornerPoints[1] = id + 1; // rb
            _gen.properties[i].cornerPoints[2] = id + zPointNumber + 1; // rt
            _gen.properties[i].cornerPoints[3] = id + zPointNumber; // lt
        }
    }

    void initializeGridGraph()
    {
        graph = new AdjacencyGraph<VertexProperty, TaggedEdge<VertexProperty, EdgeProperty>>();

        int vID = 0, eID = 0;
        int xDim = xPointNumber, zDim = zPointNumber;

        for (int i = 0; i < xDim; i++)
        {
            for (int j = 0; j < zDim; j++)
            {
                VertexProperty v = new VertexProperty();
                v.index = vID;
                float x = _shift * ((float)i - ((float)xDim - 1.0f) / 2.0f);
                float y = 0;
                float z = _shift * ((float)j - ((float)zDim - 1.0f) / 2.0f);
                v.position = new Vector3(x, y, z);
                v.fixedPosition = new Vector3(x, y, z);
                graph.AddVertex(v);
                vID++;
            }
        }

        // Add horizontal edges
        for (int i = 0; i < xDim; i++)
        {
            for (int j = 1; j < zDim; j++)
            {
                int id = i * zDim + j - 1;
                int idNext = i * zDim + j;
                VertexProperty v = graph.Vertices.ElementAt(id);
                VertexProperty vNext = graph.Vertices.ElementAt(idNext);
                EdgeProperty ep = new EdgeProperty();

                TaggedEdge<VertexProperty, EdgeProperty> edgeF =
                    new TaggedEdge<VertexProperty, EdgeProperty>(v, vNext, ep);
                edgeF.Tag.index = eID;
                edgeF.Tag.source = v;
                edgeF.Tag.target = vNext;
                graph.AddEdge(edgeF);
                eID++;

                TaggedEdge<VertexProperty, EdgeProperty> edgeB =
                    new TaggedEdge<VertexProperty, EdgeProperty>(vNext, v, ep);
                edgeB.Tag.index = eID;
                edgeB.Tag.source = vNext;
                edgeB.Tag.target = v;
                graph.AddEdge(edgeB);
                eID++;
            }
        }

        // Add vertical edges
        var rand = new System.Random();
        for (int j = 0; j < zDim; j++)
        {
            for (int i = 1; i < xDim; i++)
            {
                int id = (i - 1) * zDim + j;
                int idNext = i * zDim + j;
                VertexProperty v = graph.Vertices.ElementAt(id);
                VertexProperty vNext = graph.Vertices.ElementAt(idNext);
                EdgeProperty ep = new EdgeProperty();

                TaggedEdge<VertexProperty, EdgeProperty> edgeF =
                    new TaggedEdge<VertexProperty, EdgeProperty>(v, vNext, ep);
                edgeF.Tag.index = eID;
                edgeF.Tag.source = v;
                edgeF.Tag.target = vNext;
                graph.AddEdge(edgeF);
                eID++;

                TaggedEdge<VertexProperty, EdgeProperty> edgeB =
                    new TaggedEdge<VertexProperty, EdgeProperty>(vNext, v, ep);
                edgeB.Tag.index = eID;
                edgeB.Tag.source = vNext;
                edgeB.Tag.target = v;
                graph.AddEdge(edgeB);
                eID++;
            }
        }
    }

    private void updateControlPointPosition()
    {
        // get the position of game objects from the unity ui
        for (int i = 0; i < _controlPoints.Count; i++)
        {
            VertexProperty v = graph.Vertices.ElementAt(i);
            v.position = _controlPoints[i].transform.position;
        }
    }

    private void updateMeshPosition()
    {
        for (int i = 0; i < _mesh.vertices.Length; i++)
        {
            int lb = _gen.properties[i].cornerPoints[0]; // lb
            int rb = _gen.properties[i].cornerPoints[1]; // rb
            int rt = _gen.properties[i].cornerPoints[2]; // rt
            int lt = _gen.properties[i].cornerPoints[3]; // lt

            Vector3 fixedC = _gen.fixedVertices[i];

            Vector3[] oldCorners = // lb, rb, rt, lt
            {
                graph.Vertices.ElementAt(lb).fixedPosition,
                graph.Vertices.ElementAt(rb).fixedPosition,
                graph.Vertices.ElementAt(rt).fixedPosition,
                graph.Vertices.ElementAt(lt).fixedPosition
            };

            Vector3[] corners = // lb, rb, rt, lt
            {
                graph.Vertices.ElementAt(lb).position,
                graph.Vertices.ElementAt(rb).position,
                graph.Vertices.ElementAt(rt).position,
                graph.Vertices.ElementAt(lt).position
            };

            Vector3[] oldMiddles = // bottom, right, top, left
            {
                new Vector3(0, 0, 0),
                new Vector3(0, 0, 0),
                new Vector3(0, 0, 0),
                new Vector3(0, 0, 0)
            };

            Vector3[] middles = // bottom, right, top, left
            {
                new Vector3(0, 0, 0),
                new Vector3(0, 0, 0),
                new Vector3(0, 0, 0),
                new Vector3(0, 0, 0)
            };

            // calculate old bottom, right, top, left
            for (int j = 0; j < 4; j++)
            {
                Vector3 a = oldCorners[j];
                Vector3 b = oldCorners[(j + 1) % 4];
                oldMiddles[j] = Vector3.Dot(fixedC - a, b - a) / (b - a).magnitude / (b - a).magnitude * (b - a) + a;
                // Debug.Log("oldMiddles[j] = " + oldMiddles[j]);
            }

            // calculate middles
            for (int j = 0; j < 4; j++)
            {
                Vector3 oa = oldCorners[j];
                Vector3 ob = oldCorners[(j + 1) % 4];
                Vector3 a = corners[j];
                Vector3 b = corners[(j + 1) % 4];

                //Debug.Log("oldMiddles[" + j + "] = " + oldMiddles[j]);
                float ratio = (oldMiddles[j] - oa).magnitude / (ob - oa).magnitude;
                middles[j] = ratio * (b - a) + a;
            }

            // Line AB represented as a1x + b1y = c1
            double a1 = middles[2].z - middles[0].z;
            double b1 = middles[0].x - middles[2].x;
            double c1 = a1 * (middles[0].x) + b1 * (middles[0].z);

            // Line CD represented as a2x + b2y = c2
            double a2 = middles[3].z - middles[1].z;
            double b2 = middles[1].x - middles[3].x;
            double c2 = a2 * (middles[1].x) + b2 * (middles[1].z);

            double determinant = a1 * b2 - a2 * b1;
            double x = 0, z = 0;

            if (determinant == 0)
            {
                Debug.Log("something is wrong here!");
            }
            else
            {
                x = (b2 * c1 - b1 * c2) / determinant;
                z = (a1 * c2 - a2 * c1) / determinant;
            }

            // 0.01f let us see the mesh properly
            Vector3 position = new Vector3((float)x, 0.01f, (float)z);

            // update mesh vertex position
            Mesh mesh = GetComponent<MeshFilter>().mesh;
            Vector3[] vertices = mesh.vertices;
            vertices[i] = position;
            //Debug.Log("mesh size = " + mesh.vertices.Length);
            mesh.vertices = vertices;
            mesh.RecalculateNormals();
        }
    }

    private void OnDrawGizmos()
    {
        if (graph != null && _mesh != null)
        {
            // draw the graph
            // Debug.Log("VertexNum: " + graph.VertexCount);
            // Debug.Log("EdgeNum: " + graph.EdgeCount);
            // Debug.Log(graph.Vertices.Count());

            // draw edges
            foreach (TaggedEdge<VertexProperty, EdgeProperty> edge in graph.Edges)
            {
                Gizmos.color = edge.Tag.color;
                Gizmos.DrawLine(edge.Source.position, edge.Target.position);
            }

            // draw vertices
            foreach (VertexProperty vertex in graph.Vertices)
            {
                Gizmos.color = vertex.color;
                Gizmos.DrawSphere(vertex.position, 0.25f);
            }
        }
    }
}
```

---
# External Resources

