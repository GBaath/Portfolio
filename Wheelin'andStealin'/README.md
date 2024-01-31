# *Wheelin' and Stealin**

## *Description*

This page contains some fo the work I contributed with in the making of Wheelin' and Stealin'; a 7 week VR gameproject made by a team of 8. 
The gameplay consists of exploring dungeons, and shooting enemies, while controlling a wheelchair.
The full repo can be found [Here](https://github.com/GBaath/VR)

----

## *Main Contributions*

----

## **Wheelchair controls**

Luckily, really early on in the projects lifespan, I managed to find an existing wheelchair controller made by [**justinmajetich**](https://github.com/justinmajetich/vr-wheelchair) which I could easily get started with,
most of the tweaks I made physics values, rotation locking, and collider smoothing. Needless to say, it took alot of *n*a*u*s*e*a trying to balance all the numbers.

<img src="Images\wheelchairaction.gif" width="50%"/>

----

## **Room Generation**

*note: Most of the functionality are meant for use with prefabs and unity events, so much of the code consists of seemingly disconnected functions.*

The room generation algorithm can be broken down into 3 parts: Spawning a basemodule, filling said modules connection points, and spawning decor.
<img src="Images\DungeonGen_demo.gif" width="100%"/>

The base module contains info about the room prefabs connection points, avaliable decor sets, as well as spawnpoints for loot and enemies.
These variables are set from the prefab asset, making it easy for designers to create rooms fast. Since the connectors provide an element of random rotations and bonus modules, you can get a lot of contont to explore from less man-made assets.

<table>
  <tr>
    <td width="50%"><img src="Images\base.PNG"/></td>
    <td width="50%"><img src="Images\roomstructure.PNG"/></td>
  </tr>
</table>
<details>
<summary>RoomBase</summary>

 ```csharp

using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using System.Linq;

public class RoomBase : MonoBehaviour
{
    [Tooltip("NEEDS TO HAVE LEAST 2 CONNECTIONS")]
    [SerializeField] Transform connectionspointsHolder;
    [SerializeField]List<ConnectionsPoint> connectionsPoints;
    [SerializeField] Transform roomBase;
    [HideInInspector] public RoomEntrance entrance;
    List<int> freeIndexes = new List<int>();

    [SerializeField] Transform enemyHolder;

    [Tooltip("Leave empty for all")]
    public List<PropResource.PropType> acceptedTypes;
    private Material customRoomMaterial;
    [SerializeField]private MaterialPropertyBlock customPropertyBlock;

    [SerializeField] private List<Transform> rendererHolders;

    //info about loded objects in room
    public List<GameObject> enemies, loot, props, propSets, switchSpawnPoints;

    [SerializeField]private bool enemiesLoaded, lootLoaded, propsLoaded;
    int indexE = 0, indexL = 0, indexP = 0;

    private void Start()
    {
        customRoomMaterial = GameManager.instance.assetLoader.dungeonMaterials[Random.Range(0, GameManager.instance.assetLoader.dungeonMaterials.Length)];

        //link connections point transforms
        for (int j = 0; j < connectionspointsHolder.childCount; j++)
        {
            connectionsPoints.Add(connectionspointsHolder.GetChild(j).GetComponent<ConnectionsPoint>());
            connectionsPoints[j].baseConnection = this;
        }
        int i = Random.Range(0, propSets.Count);
        propSets[i].SetActive(true);
        //this might break ad add parent to list also
        for (int j = 0; j < propSets[i].transform.childCount; j++)
        {
            props.Add(propSets[i].transform.GetChild(j).gameObject);
        }
        for (int j = 0; j < enemyHolder.childCount; j++)
        {
            enemies.Add(enemyHolder.GetChild(j).gameObject);
        } 

        SetEntryConnection();
        SpawnConnections();

        foreach(var cp in connectionsPoints)
        {
            rendererHolders.Add(cp.transform);
        }
        //apply material to meshes
        foreach(var holder in rendererHolders)
        {
            foreach (var mr in holder.GetComponentsInChildren<MeshRenderer>())
            {
                //mr.GetComponent<Renderer>().SetPropertyBlock()
                mr.material.mainTexture = customRoomMaterial.mainTexture;
            }
        }
        foreach (var mr in entrance.GetComponentsInChildren<MeshRenderer>())
        {
            mr.material.mainTexture = customRoomMaterial.mainTexture;
        }
    }
    //places the roombase correctly for random connection point
    void SetEntryConnection()
    {
        int r = Random.Range(0, connectionsPoints.Count);

        //does roombase have a set entrancepoint?
        foreach (ConnectionsPoint c in connectionsPoints.Where(cp => !cp.GetComponent<ConnectionsPoint>().isEntrance))
        {
            r = connectionsPoints.IndexOf(c);
        }

        connectionsPoints[r].isUsed = true;

        //connect to rotationpoint and connect
        roomBase.parent = connectionsPoints[r].transform;

        foreach (ConnectionsPoint c in connectionsPoints.Where(cp => !cp.GetComponent<ConnectionsPoint>().isUsed))
        {
            c.transform.parent = roomBase;
            //save indexes of free connectionpoints
            freeIndexes.Add(connectionsPoints.IndexOf(c));
        }

        //rotation transform magic for correct pos
        connectionsPoints[r].transform.position = entrance.spawnpoint.position;
        connectionsPoints[r].transform.rotation = entrance.spawnpoint.rotation;
    }
    public void SpawnConnections()
    {

        //get random free point
        int index = freeIndexes[Random.Range(0, freeIndexes.Count)];

        //spawn exit first
        connectionsPoints[index].SpawnExitModule(out entrance.nextRoomEntrance);

        //spawn new module on free space from roommanager
        foreach (ConnectionsPoint c in connectionsPoints.Where(cp => !cp.GetComponent<ConnectionsPoint>().isUsed))
        {
            c.SpawnRandomModule();
        }
    }
    public void LoadTrigger()
    {
        entrance.UnloadPrev();
        entrance.LoadNext();
        entrance.loadDoor.Lock(true);
    }
    public void LoadRoomcontent()
    {
        if (!enemiesLoaded)
        {
            Invoke(nameof(LoadNextEnemy), .1f);
            return;
        }

        if (!lootLoaded)
        {
            Invoke(nameof(LoadNextLoot), .1f);
            return;
        }

        if (!propsLoaded)
        {
            Invoke(nameof(LoadNextProp), .1f);
            return;
        }

        entrance.nextRoomEntrance.loadDoor.Lock(false);
    }

    public void LoadNextEnemy()
    {
        if (indexE >= enemies.Count)
        {
            enemiesLoaded = true;
            LoadRoomcontent();
            return;
        }
        enemies[indexE].SetActive(true);
        indexE++;
        LoadRoomcontent();
    }
    public void LoadNextLoot()
    {
        if (indexL >= loot.Count)
        {
            lootLoaded = true;
            LoadRoomcontent();
            return;
        }

        loot[indexL].SetActive(true);
        indexL++;
        LoadRoomcontent();
    }
    public void LoadNextProp()
    {
        if (indexP >= props.Count)
        {
            propsLoaded = true;
            LoadRoomcontent();
            return;
        }
        props[indexP].SetActive(true);

        if(customRoomMaterial != null)
        {
            if (props[indexP].GetComponent<MeshRenderer>())
                props[indexP].GetComponent<MeshRenderer>().sharedMaterial = customRoomMaterial;
        }
        indexP++;
        LoadRoomcontent();
    }
}

```
</details>

The basemodules are then connected to eachother, as well as sub modules, using connection points placed on the prafab (the blue dots in the image above). 
As you enter a newly generated room, a new one is generated behind a new door, and the previous one gets unloaded.

<table>
  <tr>
    <td width="50%"><img src="Images\connection.PNG"/></td>
    <td width="50%"><img src="Images\doorloader.PNG"/></td>
  </tr>
</table>
<details>
<summary>RoomEntrance</summary>

 ```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class RoomEntrance : MonoBehaviour
{
    //roombase needs custom point because of transform stats
    public Transform spawnpoint;
    public RoomEntrance previousRoom;

    //used from other scripts when loading and unloading
    public Door loadDoor;

    public RoomBase roomBase;

    [HideInInspector] public RoomEntrance nextRoomEntrance;



    public void UnloadPrev()
    {
        try //fix for spawner room
        {
            //previousRoom.roomBase.gameObject.SetActive(false);
            Destroy(previousRoom.roomBase.gameObject);
            Destroy(previousRoom.gameObject);
        }
        catch { Destroy(previousRoom.gameObject); }

        //previousRoom.gameObject.SetActive(false);
  
        GameManager.instance.dw.DestroyWeaponsOnGround();
        //todo destroy and pool
    }
    public void LoadNext()
    {
        nextRoomEntrance.SpawnRoomBase();
    }
    public void SpawnRoomBase()
    {
        GameObject _new;
        if (GameManager.instance.roomManager.nextIsCorridor)
        {
            _new = Instantiate(GameManager.instance.roomManager.GetNewCorridor(), spawnpoint.position, Quaternion.identity);
            roomBase = _new.GetComponent<RoomBase>();
            roomBase.entrance = this;
        }
        else
        {
            ////spawn entrance module and set variables, correct transform connection is set from roombase
            _new = Instantiate(GameManager.instance.roomManager.GetNewRoomBase(),spawnpoint.position,Quaternion.identity);
            roomBase = _new.GetComponent<RoomBase>();
            roomBase.entrance = this;
        }

        roomBase.LoadRoomcontent();
        loadDoor.Lock(false);

        GameManager.instance.roomManager.roomsPassed++;
        GameManager.instance.roomManager.nextIsCorridor = !GameManager.instance.roomManager.nextIsCorridor;
    }

}
```
</details>
