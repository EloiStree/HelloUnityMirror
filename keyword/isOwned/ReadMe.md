
Example of script to request ownership:
```cs
using System.Collections.Generic;
using Mirror;
using UnityEngine;
using UnityEngine.Events;

public class HexaChessMirrorMono_ClaimAuthorityOfMoving : NetworkBehaviour
{
    public static List<HexaChessMirrorMono_ClaimAuthorityOfMoving> Instances { get; } = new();

    [Header("Target Object")]
    [SerializeField] private NetworkIdentity m_toAffect;

    [SyncVar(hook = nameof(OnOwnerChanged))]
    public uint currentOwnerNetId = 0;

    private void Awake()
    {
        if (m_toAffect == null)
            m_toAffect = GetComponent<NetworkIdentity>();
    }

    private void OnEnable()
    {
        Instances.Add(this);
    }
    private void OnDisable()
    {
        Instances.Remove(this);
    }

    [ContextMenu("Claim All Chess Pieces")]
    public void ClaimAllChessPiece() { 
    
        foreach (var piece in Instances)
        {
            piece.ClaimAuthority();
        }
    }

    [Client]
    public void OnInteracted()
    {
        ClaimAuthority();
    }

    public NetworkIdentity GetLocalPlayerNetworkIdentity()
    {
        return NetworkClient.localPlayer;
    }

    [ContextMenu("Claim Chess Piece")]
    public void ClaimAuthority()
    {
        if (m_toAffect == null) return;
        NetworkIdentity localPlayer = GetLocalPlayerNetworkIdentity();
        if (localPlayer == null) return;
        if (currentOwnerNetId == localPlayer.netId) return;
        ClaimAuthorityFromClientNetworkId(localPlayer.netId);
    }

    [Command(requiresAuthority = false)]
    private void ClaimAuthorityFromClientNetworkId(uint playerNetId)
    {
        if (m_toAffect == null) return;
        if (!NetworkServer.spawned.TryGetValue(playerNetId, out NetworkIdentity playerIdentity))return;
        NetworkConnectionToClient targetConnection = playerIdentity.connectionToClient;
        if (targetConnection == null)return;
        m_toAffect.RemoveClientAuthority();
        m_toAffect.AssignClientAuthority(targetConnection);
        currentOwnerNetId = playerNetId;   
    }

    public UnityEvent<uint, uint> m_onOwnerChangedFromTo = new UnityEvent<uint, uint>();
    public UnityEvent <bool> m_onClaimState;
    private void OnOwnerChanged(uint oldOwner, uint newOwner)
    {
        m_onOwnerChangedFromTo?.Invoke(oldOwner, newOwner);
        bool isMine = newOwner == (NetworkClient.localPlayer?.netId ?? 0);
        m_onClaimState?.Invoke(isMine);
    }
   
    public bool HasAuthority()
    {
        if (m_toAffect == null) return false;
        return m_toAffect.isOwned;
    }
}
```
