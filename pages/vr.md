---
# permalink: /about/
layout: single
title: "VR"
classes: wide
header:
  image: /assets/images/teaser/teaser.png
  caption: "Image credit: [**Yun**](http://yun-vis.net)"
last_modified_at: 2026-05-26
---

# Meta Quest Setup

Meta Quest 3 #5

1. Create spacing for glasses users: Pess the two buttons on both sides of the headset (one at a time) and slowly pull the facial interface away from the lens. (This is shown in the introduction video)
2. Prepare the zip file of the sample project

## Unity Packages

1. XR Plugin Management (The Manager)
   - What it does: It acts as the master switchboard for your project's XR settings.
   - Why you need it: It allows you to check a box to turn on "OpenXR" for standalone builds (like Quest) while keeping it off or switching to a different provider for PC or WebGL builds.

2. OpenXR Plugin (The Translator)
   - What it does: It talks directly to the headset's operating system using the universal OpenXR language.
   - Why you need it: Without it, Unity cannot communicate with hardware. It handles tracking, rendering alignment, and brings in the raw button data from your controllers.
   - In Unity, Multi-pass rendering means the engine renders the entire game world twice—once for the left eye, and once for the right eye.It is the older, less optimized way to render VR graphics, but it offers the highest compatibility with complex shaders.

3. XR Interaction Toolkit (The Gameplay)
   - What it does: It translates raw data into actual VR gameplay mechanics.
   - Why you need it: OpenXR only tells Unity where the controller is and if a button is pressed. The XR Interaction Toolkit uses that data to let the player grab objects, teleport, climb, and interact with menus without you having to code those systems from scratch.
   - XRI Default Input Actions 
   - 
## Open Questions

- [Update Meta Quest software via a USB-C cable (still need to check)](https://www.meta.com/en-gb/help/quest/software_update/)
- Do we need windows build support?

# BallSpawner

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

# DuckManager

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class DuckManager : MonoBehaviour
{
    // [SerializeField] private Transform[] possibleDuckSpawns;
    [SerializeField] private GameObject[] duckVariants;

    void Start()
    {
        SpawnDuck();
    }

    public void SpawnDuck()
    {
        // Transform spawnPos = possibleDuckSpawns[Random.Range(0, possibleDuckSpawns.Length)];
        GameObject duckVariant = duckVariants[Random.Range(0, duckVariants.Length)];
        duckVariant.transform.position = transform.position;
        // y, z axes are inverted?
        duckVariant.transform.Translate(new Vector3(Random.Range(-3.0f, 3.0f), Random.Range(-3.0f, 3.0f), 0.0f));
        // duckVariant.transform.Translate(Random.Range(-1.0f, 1.0f), 0.0f, Random.Range(-1.0f, 1.0f));
        // Debug.Log("Spawning duck at rotation: " + duckVariant.transform.localRotation);
        duckVariant.transform.localRotation = Quaternion.AngleAxis(Random.Range(0, 360), Vector3.forward);
        GameObject newObject = Instantiate(duckVariant, transform);
        newObject.SetActive(true);
    }
}
```

# DestroyOnCollideWithTag

```csharp
using System.Collections;
using System.Collections.Generic;
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

