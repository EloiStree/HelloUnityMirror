

Pour bouger une piece dans un scene, il faut que vous la possedier.
Si c est Spawned par vous, c est vous le posseseur par defaut.

Mais dans le cas d une piece d echec.
Il vous faut la reclamer avant de la bouger.
Car un seul client peu la bouger a la fois.

Voici un code que jai  utiliser pour la demo des jeux d echec.


```cs
using System.Collections.Generic;
using Mirror;
using UnityEngine;

// On ajouter un script a une piece aec le NetworkBehaviour
public class HexaChessMirrorMono_ClaimAuthorityOfMoving : NetworkBehaviour
{

    // Specifions qu'elle piece peut etre claim
    [Header("Target Object")]
    [SerializeField] private NetworkIdentity m_toAffect;

    public void Reset()
    {
        // Normalement c est celle du script actuelle.
        m_toAffect = GetComponent<NetworkIdentity>();
    }

    // On ne change le priorietaire que si il l est pas.
    // Il faut donc que le server donne l'information de a qui elle appartien
    // En utilisant le uint NetId de mirror utiliser pour index les joueurs.
    [SyncVar(hook = nameof(OnOwnerChanged))]
    public uint currentOwnerNetId = 0;

    // On interacted est une methode du  Open XR Toolkit.
    // Cela permet de savoir si on l utilise.
    public void OnInteracted()
    {
        // demande depuis chez un client de gagner la prorieter de l'object
        // Si on interagit avec elle
        ClaimAuthority();
    }

    //Preparons la demande chez le client
    [ContextMenu("Force Claim Authority")]
    public void ClaimAuthority()
    {
        if (m_toAffect == null) return;
        // On demande de recuprer le int NetId du joueru qui est considrer comme local a cette application/device
        NetworkIdentity localPlayer = LookInSceneForLocalPlayer();
        // Si il y en a pas, on degage 😉
        if (localPlayer == null)       
            return;
        // Si l object est deja posseder par le jouer actuelle.
        // On as pas besoin de demander au server
        if (currentOwnerNetId == localPlayer.netId)
            return;
        // Allons demander au sever avec [Command]
        ClaimAuthorityFromClientNetworkId(localPlayer.netId);
    }

    //Dans le nouveau Mirror vous pouvez savoir qui est local comme ceci.
    public NetworkIdentity LookInSceneForLocalPlayer()
    {
        return NetworkClient.localPlayer;
    }

    // RequireesAuthority permet de dit que n import que joueur peu appeler la methode
    // En tout cas sur un objet de la scene.
    // https://mirror-networking.gitbook.io/docs/manual/guides/communications/remote-actions
    [Command(requiresAuthority = false)]
    // On envoie au server le number d index multijoueur de notre jouer local.
    // Pour lui dire donne lui l authoriter de l object.
    private void ClaimAuthorityFromClientNetworkId(uint playerNetId)
    {
        if (m_toAffect == null) return;
        // Un petite verification qu il existe bien ce joueur
        if (!NetworkServer.spawned.TryGetValue(playerNetId, out NetworkIdentity playerIdentity))
        {
            Debug.LogWarning($"ClaimAuthority: No player found with netId {playerNetId}.");
            return;
        }
        // Et une petit verification que la connection existe toujours
        NetworkConnectionToClient targetConnection = playerIdentity.connectionToClient;
        if (targetConnection == null)
        {
            Debug.LogWarning($"ClaimAuthority: Player {playerNetId} has no client connection.");
            return;
        }
    
        // On retire les authoirite deja presentep par prudence
        m_toAffect.RemoveClientAuthority();
        // Et on donne l authoriter du joueur trouver pour notre gameobject.
        m_toAffect.AssignClientAuthority(targetConnection);
        // Pour eviter le spam on notifie tout le monde que il y a eu un changement de proprietaire via le SyncVar
        currentOwnerNetId = playerNetId; 
        
    }

    // Le SyncVar du proprietaire ayant changer on peu le notifier par Deub Log si necessaire.
    private void OnOwnerChanged(uint oldOwner, uint newOwner)
    {
        bool isMine = newOwner == (NetworkClient.localPlayer?.netId ?? 0);
    }
    
    public bool HasAuthority()
    {
        if (m_toAffect == null) return false;
        var nb = m_toAffect.GetComponent<NetworkBehaviour>();
        if (nb != null)
            return nb.isOwned;
        return m_toAffect.isOwned;
    }
}
```