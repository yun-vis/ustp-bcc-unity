---
# permalink: /about/
layout: single
title: "Animation"
classes: wide
header:
  image: /assets/images/teaser/teaser.png
  caption: "Image credit: [**Yun**](http://yun-vis.net)"
last_modified_at: 2023-05-03
---

# Interpolation

## Ease-In Ease-Out Function

Network.cs
```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using System.Linq;
using QuikGraph;

// self-defined namespace
using Property;
using QuikGraph.Algorithms.Observers;
using QuikGraph.Algorithms.ShortestPath;
using AnimationEaseInOut;

public class Network : MonoBehaviour
{
    // graph
    private AdjacencyGraph<VertexProperty, TaggedEdge<VertexProperty, EdgeProperty>> _graph;

    // variables for path finding
    private int _sourceID = 9;
    private int _targetID = 2;

    private List<Vector3> _coordList = new List<Vector3>();

    // variables for ease-in ease-out functions
    private bool _isMoveForward = true;
    private int _currentCoordID = 0;
    private Vector3 _targetCoord;
    private float _rotateSpeed = 3.0f;
    private float _accumulatedTime = 0.0f;

    void Awake()
    {
        // initialization
        InitializeGraph();
    }

    // Start is called before the first frame update
    void Start()
    {
        // find the path
        FindDijkstraPath();

        // move game object position to the source position
        if (_coordList.Count > 1)
        {
            transform.position = _coordList[0];
            _targetCoord = _coordList[1];
        }
    }

    // Update is called once per frame
    void Update()
    {
        // source == target
        if (_coordList.Count <= 1)
            return;

        // move the object forward
        if (_isMoveForward)
        {
            WalkForward();

            if (_currentCoordID == _coordList.Count - 1)
            {
                _isMoveForward = !_isMoveForward;
            }
        }
        // move the object backward
        else
        {
            WalkBackward();

            if (_currentCoordID == 0)
            {
                _isMoveForward = !_isMoveForward;
            }
        }
    }

    void WalkForward()
    {
        // rotate toward the target 
        transform.forward = Vector3.RotateTowards(transform.forward, _targetCoord - transform.position,
            _rotateSpeed * Time.deltaTime, 0.0f);

        // move the obj toward the target position
        // transform.position = Vector3.MoveTowards(transform.position, targetCoord, moveSpeed * Time.deltaTime);
        // transform.position = Vector3.MoveTowards(transform.position, targetCoord, moveSpeed * Time.deltaTime);
        _accumulatedTime += Time.deltaTime;

        // Debug.Log("_currentCoordID = " + _currentCoordID);
        transform.position =
            EaseInOut.MoveTowardSmoothstep(_coordList[_currentCoordID], _targetCoord, _accumulatedTime);

        // move to the next edge segment
        if (transform.position == _targetCoord)
        {
            _currentCoordID++;
            if (_currentCoordID <= _coordList.Count - 2)
            {
                _targetCoord = _coordList[_currentCoordID + 1];
                _accumulatedTime = 0;
            }
            else
            {
                _targetCoord = _coordList[_coordList.Count - 2];
                _accumulatedTime = 0;
            }
        }
    }

    void WalkBackward()
    {
        // rotate toward the target 
        transform.forward = Vector3.RotateTowards(transform.forward, _targetCoord - transform.position,
            _rotateSpeed * Time.deltaTime, 0.0f);

        // move the obj toward the target position
        // transform.position = Vector3.MoveTowards(transform.position, targetCoord, moveSpeed * Time.deltaTime);

        _accumulatedTime += Time.deltaTime;
        transform.position =
            EaseInOut.MoveTowardSmoothstep(_coordList[_currentCoordID], _targetCoord, _accumulatedTime);

        // move to the next edge segment
        if (transform.position == _targetCoord)
        {
            _currentCoordID--;
            if (_currentCoordID >= 1)
            {
                _targetCoord = _coordList[_currentCoordID - 1];
                _accumulatedTime = 0;
            }
            else
            {
                _targetCoord = _coordList[1];
                _accumulatedTime = 0;
            }
        }
    }

    void FindDijkstraPath()
    {
        // initialization
        VertexProperty source = _graph.Vertices.ElementAt(_sourceID);
        VertexProperty target = _graph.Vertices.ElementAt(_targetID);

        // delegate
        Func<TaggedEdge<VertexProperty, EdgeProperty>, double> edgeCost = GetEdgeDistance;

        // Create the algorithm instance
        var dijkstra = new DijkstraShortestPathAlgorithm<VertexProperty,
            TaggedEdge<VertexProperty, EdgeProperty>>(_graph, edgeCost);

        // Create the observer
        var vpro = new VertexPredecessorRecorderObserver<VertexProperty, TaggedEdge<VertexProperty, EdgeProperty>>();

        // Compute and record shortest paths
        using (vpro.Attach(dijkstra))
        {
            dijkstra.Compute(source);
        }

        // vpro can create all the shortest path from source in the graph
        if (vpro.TryGetPath(target, out IEnumerable<TaggedEdge<VertexProperty, EdgeProperty>> shortestpath))
        {
            bool isFirst = true;
            VertexProperty prevT = new VertexProperty();;
            
            // extract the shortest path
            foreach (TaggedEdge<VertexProperty, EdgeProperty> edge in shortestpath)
            {
                VertexProperty s = edge.Tag.source;
                VertexProperty t = edge.Tag.target;

                // Debug.Log("(s,t) = (" + s.index + "," + t.index + ")");
                edge.Tag.selected = true;
                if (isFirst)
                {
                    if (s.index != _sourceID)
                    {
                        VertexProperty temp = s;
                        s = t;
                        prevT = t = temp;
                    }
                    else
                    {
                        prevT = t;
                    }

                    // Debug.Log("(s,t) = (" + s.index + "," + t.index + ")");
                    // Debug.Log("prevT = " + prevT.index);

                    _coordList.Add(s.position);
                    _coordList.Add(t.position);
                    s.selected = true;
                    t.selected = true;
                    isFirst = false;
                }
                else
                {
                    if (prevT.index != s.index)
                    {
                        VertexProperty temp = s;
                        s = t;
                        prevT = t = temp;
                    }
                    else
                    {
                        prevT = t;
                    }

                    // Debug.Log("(s,t) = (" + s.index + "," + t.index + ")");

                    _coordList.Add(t.position);
                    t.selected = true;
                }
            }
        }
    }

    private double GetEdgeDistance(TaggedEdge<VertexProperty, EdgeProperty> edge)
    {
        return edge.Tag.distance;
    }

    void InitializeGraph()
    {
        _graph = new AdjacencyGraph<VertexProperty, TaggedEdge<VertexProperty, EdgeProperty>>();

        // variables for initialization
        int vID = 0, eID = 0;
        int xDim = 5, zDim = 3;
        float shift = 4.0f;

        // create vertices
        for (int i = 0; i < xDim; i++)
        {
            for (int j = 0; j < zDim; j++)
            {
                VertexProperty v = new VertexProperty();
                v.index = vID;
                float x = shift * (i - ((float)xDim - 1.0f) / 2.0f);
                float y = 0;
                float z = shift * (j - ((float)zDim - 1.0f) / 2.0f);
                v.position = new Vector3(x, y, z);
                vID++;

                // add the vertex to the graph
                _graph.AddVertex(v);
            }
        }

        // create edges
        // Add horizontal edges
        for (int i = 0; i < xDim; i++)
        {
            for (int j = 1; j < zDim; j++)
            {
                int id = i * zDim + j - 1;
                int idNext = i * zDim + j;

                // getting source and target vertices
                VertexProperty v = _graph.Vertices.ElementAt(id);
                VertexProperty vNext = _graph.Vertices.ElementAt(idNext);
                EdgeProperty ep = new EdgeProperty();

                // create an edge
                TaggedEdge<VertexProperty, EdgeProperty> edgeF = new TaggedEdge<VertexProperty,
                    EdgeProperty>(v, vNext, ep);
                // adding edge property
                edgeF.Tag.index = eID;
                edgeF.Tag.source = v;
                edgeF.Tag.target = vNext;
                _graph.AddEdge(edgeF);
                eID++;

                TaggedEdge<VertexProperty, EdgeProperty> edgeB = new TaggedEdge<VertexProperty,
                    EdgeProperty>(vNext, v, ep);
                // adding edge property
                edgeB.Tag.index = eID;
                edgeB.Tag.source = vNext;
                edgeB.Tag.target = v;
                _graph.AddEdge(edgeB);
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
                VertexProperty v = _graph.Vertices.ElementAt(id);
                VertexProperty vNext = _graph.Vertices.ElementAt(idNext);
                EdgeProperty ep = new EdgeProperty();

                // Add the edge to the graph
                TaggedEdge<VertexProperty, EdgeProperty> edgeF = new TaggedEdge<VertexProperty,
                    EdgeProperty>(v, vNext, ep);
                edgeF.Tag.index = eID;
                edgeF.Tag.source = v;
                edgeF.Tag.target = vNext;
                _graph.AddEdge(edgeF);
                eID++;

                TaggedEdge<VertexProperty, EdgeProperty> edgeB = new TaggedEdge<VertexProperty,
                    EdgeProperty>(vNext, v, ep);
                edgeB.Tag.index = eID;
                edgeB.Tag.source = vNext;
                edgeB.Tag.target = v;
                _graph.AddEdge(edgeB);
                eID++;
            }
        }
    }

    private void OnDrawGizmos()
    {
        if (_graph == null)
        {
            return;
        }

        // Draw the graph
        // Draw vertices
        foreach (VertexProperty vertex in _graph.Vertices)
        {
            if (vertex.selected)
            {
                Gizmos.color = Color.red;
            }
            else
            {
                Gizmos.color = Color.black;
            }

            Gizmos.DrawSphere(vertex.position, 0.125f);
        }

        // Draw edges
        foreach (TaggedEdge<VertexProperty, EdgeProperty> edge in _graph.Edges)
        {
            if (edge.Tag.selected)
            {
                Gizmos.color = Color.blue;
            }
            else
            {
                Gizmos.color = Color.black;
            }

            Gizmos.DrawLine(edge.Source.position, edge.Target.position);
        }
    }
}
```

EaseInOut.cs
```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Animations;

namespace AnimationEaseInOut
{
    public class EaseInOut
    {
        public static Vector3 MoveTowardSmoothstep(Vector3 source, Vector3 target, float t)
        {
            Vector3 position = new Vector3();

            if (t > 1) return target;
            else if (t < 0) return source;

            // easeInQuint
            // t = t * t * t * t * t;
            // t = t * t * (3.0f - 2.0f * t);
            position = t * (target - source) + source;

            return position;
        }
    }
}
```

---
# External Resources

