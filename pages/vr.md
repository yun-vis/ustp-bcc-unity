---
# permalink: /about/
layout: single
title: "OpenXR"
classes: wide
header:
  image: /assets/images/teaser/teaser.png
  caption: "Image credit: [**Yun**](http://yun-vis.net)"
last_modified_at: 2026-06-02
---

# BallSpawner

In Assets/Scripts/BallSpawner.cs,
```csharp
using UnityEngine;

public class BallSpawner : MonoBehaviour
{
    [SerializeField] private GameObject prefabToSpawn;

    private GameObject spawnedObject;

    private void Start()
    {
        prefabToSpawn.SetActive(false);
    }

    public void SpawnNewOne()
    {
        if (spawnedObject != null)
        {
            Destroy(spawnedObject);
        }

        spawnedObject = Instantiate(prefabToSpawn, transform.position, Quaternion.identity);
        spawnedObject.SetActive(true);
    }
}
```

# GlassManager

In Assets/Scripts/GlassManager.cs,
```csharp
//using System.Collections;
//using System.Collections.Generic;
using UnityEngine;

public class GlassManager : MonoBehaviour
{
    // [SerializeField] private Transform[] possibleGlassSpawns;
    [SerializeField] private GameObject[] glassVariants;

    void Awake()
    {
        SpawnGlass();
    }

    public void SpawnGlass()
    {
        Debug.Log("Spawning glass...");
        // Transform spawnPos = possibleGlassSpawns[Random.Range(0, possibleGlassSpawns.Length)];
        GameObject glassVariant = glassVariants[Random.Range(0, glassVariants.Length)];
        glassVariant.transform.position = transform.position;
        // glassVariant.transform.position = new Vector3(transform.position.x, 0.8f, transform.position.z);
        // y, z axes are inverted? Only for the rubber duck assets
        Debug.Log("Spawning glass at position: " + transform.position);

        glassVariant.transform.Translate(new Vector3(Random.Range(-1.0f, 1.0f), glassVariant.transform.position.y + 0.8f, Random.Range(-1.0f, 1.0f)));

        Debug.Log("Spawning glass at glassVariant.position: " + glassVariant.transform.position);
        // glassVariant.transform.Translate(Random.Range(-1.0f, 1.0f), 0.0f, Random.Range(-1.0f, 1.0f));
        // Debug.Log("Spawning glass at rotation: " + glassVariant.transform.localRotation);
        glassVariant.transform.localRotation = Quaternion.AngleAxis(Random.Range(-35, 35), Vector3.forward);

        GameObject newObject = Instantiate(glassVariant, glassVariant.transform.position, glassVariant.transform.rotation);
        newObject.SetActive(true);
        Debug.Log("newObject: " + newObject.name);

    }
}
```

# DestroyOnCollideWithTag

In Assets/Scripts/DestroyOnCollideWithTag.cs,
```csharp
//using System.Collections;
//using System.Collections.Generic;
using UnityEngine;
using UnityEngine.Events;

public class DestroyOnCollideWithTag : MonoBehaviour
{
    [SerializeField] private string ballTag;
    [SerializeField] private UnityEvent onDestroy;

    private void OnCollisionEnter(Collision collision)
    {
        if (collision.collider.CompareTag(ballTag))
        {
            onDestroy.Invoke();
            Destroy(gameObject);
        }
    }
}
```

# NPCBehavior

In Assets/Scripts/NPCBehavior.cs,
```csharp
using Unity.XR.CoreUtils;
using UnityEngine;

public class NPCBehavior : MonoBehaviour
{
    private Animator npcAnimator;
    private GameObject pondTelepotationArea;
    private GameObject plane;
    private bool isPondActivated = true;

    void Start()
    {
        // ground = GameObject.Find("Ground");
        pondTelepotationArea = GameObject.Find("MovementZones").gameObject;
        plane = GameObject.Find("GravelBakePlane_-0.05").gameObject;
        npcAnimator = GetComponent<Animator>();
    }

    public void ActivateNPC()
    {
        // Debug.Log("NPC Activated");
        npcAnimator.SetBool("IsActivated", true);

        isPondActivated = !isPondActivated;
        if (isPondActivated)
        {
            Debug.Log("isPondActivated: " + isPondActivated);
            // Activate the water and deactivate the rocks
            plane.transform.GetChild(0).gameObject.SetActive(true);
            plane.transform.GetChild(1).gameObject.SetActive(false);
            pondTelepotationArea.transform.GetChild(2).gameObject.SetActive(false);
        }
        else
        {
            Debug.Log("isPondActivated: " + isPondActivated);
            // Deactivate the water and activate the rocks
            plane.transform.GetChild(0).gameObject.SetActive(false);
            plane.transform.GetChild(1).gameObject.SetActive(true);
            pondTelepotationArea.transform.GetChild(2).gameObject.SetActive(true);
        }
    }

    public void DeActivateNPC()
    {
        // Debug.Log("NPC Deactivated");
        npcAnimator.SetBool("IsActivated", false);
    }
}
```

---
# External Resources

