

What is Owned, Authority  isLocalPlayer, isServer.

Et comment utiliser le singleton pour parler avec le prefab qui nous appartien en tant que joueur.



`    public override void OnStartLocalPlayer()`
`    public override void OnStartClient()`
`    public override void OnStartServer()`
`    public override void OnStartAuthority()`

NetworkClient.localPlayer

NetworkBehaviour  
`NB.isOwned`  
    


```
Un petit code pour notifier un objet local au joueur
public class HexaChessMirrorMono_HideShowIfLocal : NetworkBehaviour
{
    [SerializeField] private UnityEvent m_onIsLocalPlayerEvent;
    [SerializeField] private UnityEvent m_onIsNotLocalPlayerEvent;
    [SerializeField] private UnityEvent<bool> m_onIsLocalPlayerBoolEvent;
    [SerializeField] private UnityEvent<bool> m_onIsNotLocalPlayerBoolEvent;
    [SerializeField]
    private bool m_resultDebugIsLocalPlayer = false;
    public override void OnStartClient()
    {
        Refresh();
    }
    public override void OnStartLocalPlayer()
    {
        Refresh();
    }
    private void Refresh()
    {
        m_resultDebugIsLocalPlayer = isLocalPlayer;
        if (isLocalPlayer)
        {
            m_onIsLocalPlayerEvent?.Invoke();
            m_onIsLocalPlayerBoolEvent?.Invoke(true);
            m_onIsNotLocalPlayerBoolEvent?.Invoke(false);
        }
        else
        {
            m_onIsNotLocalPlayerEvent?.Invoke();
            m_onIsLocalPlayerBoolEvent?.Invoke(false);
            m_onIsNotLocalPlayerBoolEvent?.Invoke(true);
        }
    }
}
```

