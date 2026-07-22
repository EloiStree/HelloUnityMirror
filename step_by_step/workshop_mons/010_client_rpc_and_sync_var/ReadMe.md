
I have a video explaining how to use Mirror and RSA.

[https://github.com/EloiStree/HelloUnityMirror/tree/main/video/mirror_and_rsa_workflow](https://github.com/EloiStree/HelloUnityMirror/tree/main/video/mirror_and_rsa_workflow)

However, I'm not sure this is the topic you want to learn.

Today we will learn about **`[Command]`** and **`[SyncVar]`**.

The goal is to let a player declare:

* their integer ID,
* their public key,
* their color (once their integer ID or public key has been defined).

We won't perform any identity verification yet.

---

## Requirements

* A Unity project with **Mirror** installed.
* The package:

  * [https://github.com/EloiStree/2026_07_21_upm_unicode_watch_for_vr](https://github.com/EloiStree/2026_07_21_upm_unicode_watch_for_vr)

---

# Creating a Simple Lobby

Our goal is to create a player that can:

* choose who they want to be,
* verify their identity later,
* send a Unicode character,
* choose a color.

To do that, we first need a lobby.

Since the lobby is hosted by one of the players, that player is called the **Host**.

The Host is both:

* a **client** playing the game,
* the **server** for every other player.

To keep things simple, we'll reuse the basic lobby scene that comes with Mirror.

![alt text](image-4.png)

Copy it into your project, then close the original scene **without saving**.

![alt text](image-5.png)

---

The **Network Manager HUD** is useful for debugging while prototyping.

![alt text](image-6.png)

For example, you can:

* start as a Host,
* start as a Client,
* change the server IP that the client connects to.

![alt text](image-7.png)

---

Under the **NetworkManager**, you'll find the **KCP Transport**.

![alt text](image-8.png)

This component handles the network communication (UDP-based transport).

---

The **Network Manager** also expects a **Player Prefab**.

For a GameObject to exist on the network, it must contain a **NetworkIdentity** component.

![alt text](image-1.png)

Assign your player prefab to the Network Manager.

![alt text](image-2.png)

---

When you start the game, a player is automatically spawned.

![alt text](image-9.png)

Notice that the first player is both the **Client** and the **Server**.

![alt text](image-10.png)

---

## Creating a Network Script

Create a new script (and Assembly Definition if needed) that references Mirror.

![alt text](image-11.png)

To create a network behaviour, your class must inherit from **NetworkBehaviour**.

![alt text](image-12.png)

---

## Asking the Server to Change the Color

Let's allow the player to request a new random color.

```cs
[ContextMenu("Set New Random Color")]
public void SetNewRandomColor()
{
    Color c = new Color(Random.value, Random.value, Random.value);
    CmdSetColor(c);
}
```

The player asks the server using a **`[Command]`**.

In other words:

> "Server, I'd like my player to use this color."

```cs
[Command]
private void CmdSetColor(Color c)
{
    RpcSetColor(c);
}
```

The server then replies with a **ClientRpc**, telling every connected client about the new color.

We use a UnityEvent so designers can react to the color change without writing code.

```cs
public UnityEvent<Color> m_onColorChanged;

[ClientRpc]
private void RpcSetColor(Color c)
{
    m_onColorChanged?.Invoke(c);
}
```

The communication flow is:

**Client → `[Command]` → Server → `[ClientRpc]` → All Clients**

---

## Testing

Create a cube and add a **`UnicodeWatchMono_RelayColor`** component.

Connect the mesh renderer and the UnityEvent.

![alt text](image-13.png)

![alt text](image-14.png)

If everything works in the Editor, test it with a standalone build.

![alt text](image-15.png)

---

## Giving Players Random Positions

Let's randomly move the player for debugging.

```cs
public float m_randomSpotAtStart = 5f;

private void Start()
{
    transform.position = new Vector3(
        Random.Range(0f, m_randomSpotAtStart),
        Random.Range(0f, m_randomSpotAtStart),
        Random.Range(0f, m_randomSpotAtStart));
}
```

To make it easier to see the result, arrange the game windows like this.

![alt text](image-16.png)

The standalone build is acting as the server.

![alt text](image-17.png)

Find the player that belongs to you and change its color.

![alt text](image-18.png)

Tada! It works.

![alt text](image-19.png)

---

## Why Doesn't the Position Match?

The cube position is random, but every client sees a different position.

That's because we never synchronized the position with the other players.

You could synchronize it the same way we synchronized the color.

However, Mirror already provides a solution:

**NetworkTransform**

![alt text](image-20.png)

Unfortunately, we moved the object **before** the networking system had finished spawning it.

![alt text](image-21.png)

Instead, move it after it has joined the network.

```cs
public float m_randomSpotAtStart = 2f;

public override void OnStartClient()
{
    base.OnStartClient();

    if (!isOwned)
        return;

    transform.position = new Vector3(
        Random.Range(0f, m_randomSpotAtStart),
        Random.Range(0f, m_randomSpotAtStart),
        Random.Range(0f, m_randomSpotAtStart));
}
```

Be careful to move the object **only if you own it**.

![alt text](image-22.png)


**Code State**
```cs
using UnityEngine;
using Mirror;
using UnityEngine.Events;
namespace Eloi.UnicodeWatch
{
    [RequireComponent(typeof(NetworkIdentity))]
    public class UnicodeWatchMirrorMono_BasicPlayer : NetworkBehaviour
    {
        public UnityEvent<Color> m_onColorChanged;
        public float m_randomSpotAtStart = 2f;
        public bool m_randomColorAtStart = true;
        public override void OnStartClient()
        {
            base.OnStartClient();
            if (!isOwned)
                return;
            transform.position = new Vector3(Random.Range(0f, m_randomSpotAtStart), Random.Range(0f, m_randomSpotAtStart), Random.Range(0f, m_randomSpotAtStart));
            if (m_randomColorAtStart)
                SetNewRandomColor();
        }
        [ContextMenu("Set new Random Color")]
        public void SetNewRandomColor()
        {
            Color c = new Color(Random.value, Random.value, Random.value);
            CmdSetColor(c);
        }
        [Command]
        private void CmdSetColor(Color c)
        {
            RpcSetColor(c);
        }
        [ClientRpc]
        private void RpcSetColor(Color c)
        {
            m_onColorChanged?.Invoke(c);
        }
    }
}
```


---

# The Problem with ClientRpc

Suppose a new player joins the game later.

They never received the earlier `ClientRpc`, so they don't know what color the existing players are.

This is exactly why **`[SyncVar]`** exists.

A `SyncVar` automatically synchronizes the current value to newly connected clients, ensuring everyone sees the correct state even if they join after the change occurred.



Let's changed that.

Add a class member as SyncVar
```cs
        [SerializeField]
        [SyncVar]
        Color m_wantedWatchColor;
```

![alt text](image-23.png)


A `SyncVar` should only be changed on the server to impact all players.  
And is changeable only if the player has authority to call the server.  

Excepte if you do a [Command] without autorithy
`[Command(requiresAuthority = false)]`
Let's try it for the fun.

As we want to do action when color is change we can hook the `SyncVar`
```cs
[SerializeField]
[SyncVar(hook = nameof(HookColorChanged))]
Color m_wantedWatchColor;
private void HookColorChanged(Color oldColor, Color newColor)
{
    m_onColorChanged?.Invoke(newColor);
}
```



Let' try this full code:
```cs
using UnityEngine;
using Mirror;
using UnityEngine.Events;
namespace Eloi.UnicodeWatch
{
    [RequireComponent(typeof(NetworkIdentity))]
    public class UnicodeWatchMirrorMono_BasicPlayer : NetworkBehaviour
    {

        [SerializeField]
        [SyncVar(hook = nameof(HookColorChanged))]
        Color m_wantedWatchColor;

        public UnityEvent<Color> m_onColorChanged;
        public float m_randomSpotAtStart = 2f;
        public bool m_randomColorAtStart = true;
        
        
        
        // Player is connected to the game
        public override void OnStartClient()
        {
            base.OnStartClient();
            if (!isOwned)
                return;
            // Only if the player own this gameobject
            // We change it place (NetworkTransform) will move it everywhere
            transform.position = new Vector3(Random.Range(0f, m_randomSpotAtStart), Random.Range(0f, m_randomSpotAtStart), Random.Range(0f, m_randomSpotAtStart));
            //When designer ask a new color at start of the player
            if (m_randomColorAtStart)
                // W ask to change the color with public method
                SetNewRandomColor();
        }

        // Player ask to change color        
        [ContextMenu("Set new Random Color")]
        public void SetNewRandomColor()
        {
            Color c = new Color(Random.value, Random.value, Random.value);
            // The request is send to the server
            CmdSetColor(c);
        }
        //As it is authority require fall any can change the color of this player
        // The following code is on the server
        [Command(requiresAuthority = false)]
        private void CmdSetColor(Color c)
        {
            // We change the color
            m_wantedWatchColor = c;
            // As we are on the server and it is SyncVar
            // The change is send to all players.
        }
        // As we hook the sync var we are on the client now and the color is changed as variable.
        private void HookColorChanged(Color oldColor, Color newColor)
        {
            // But in addition, for all the device we change the color with UnityEvent
            // Of this player prefab.
            m_onColorChanged?.Invoke(newColor);
        }
    }
}
```

Should have work, but I missed something
![alt text](image-24.png)

You can still change your color.



Let's add a on click for testing.
(If Unity let's us do)
https://github.com/EloiStree/2025_06_02_upm_tick_collection/blob/main/Runtime/TickMono_OnMouseDownUp.cs
```cs
    public class UnicodeWatchMono_MouseClickEvent : MonoBehaviour, IPointerDownHandler, IPointerUpHandler
    {
        [SerializeField] private UnityEvent m_onMouseDown = new UnityEvent();
        [SerializeField] private UnityEvent m_onMouseUp = new UnityEvent();

        public void OnPointerDown(PointerEventData eventData)
        {
            m_onMouseDown?.Invoke();
        }

        public void OnPointerUp(PointerEventData eventData)
        {
            m_onMouseUp?.Invoke();
        }
    }
```

---------------

# Now let's add The Index and RSA public Key.






