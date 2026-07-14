[<img width="1322" height="836" alt="image" src="https://github.com/user-attachments/assets/1d551403-d7ff-4734-9d15-0cd673e44986" />
](https://youtu.be/oKf_EAU-ct4?t=2)
https://youtu.be/oKf_EAU-ct4?t=2


Les commandes:
[<img width="759" height="209" alt="image" src="https://github.com/user-attachments/assets/f05f2623-32ce-4af4-881a-5fb6a30f3cc3" />](https://youtu.be/oKf_EAU-ct4?t=32
)   
https://youtu.be/oKf_EAU-ct4?t=32  


Ils nous faut un cle cryptographique publique pour identifier les joueurs:
[<img width="878" height="212" alt="image" src="https://github.com/user-attachments/assets/88d6d9e5-25a9-4664-a59b-305f49c49b9f" />](https://youtu.be/oKf_EAU-ct4?t=97
)
https://youtu.be/oKf_EAU-ct4?t=97


Il nous faut un Network Identiy sur le joueur pour faire partie du reseaux Mirror:
[<img width="882" height="155" alt="image" src="https://github.com/user-attachments/assets/bd833004-0b48-43fa-a321-54c0f078da95" />](https://youtu.be/oKf_EAU-ct4?t=113)
https://youtu.be/oKf_EAU-ct4?t=113   



OnStartClient utilisable si vous heritez de NetworkBehaviour permet de dit quand le joueur est dans la partie officiellement
[<img width="673" height="259" alt="image" src="https://github.com/user-attachments/assets/8b7aadd3-2910-40d4-bc61-0f9797fc5df4" />](https://youtu.be/oKf_EAU-ct4?t=142)
https://youtu.be/oKf_EAU-ct4?t=142   

<img width="615" height="257" alt="image" src="https://github.com/user-attachments/assets/ca8027cf-4994-4f6e-9c55-834034c03eba" />
- `isServer` dit si Unity est acutellement sur le server. (Attention cela pour tout les joueurs)
- `isOwned` dit si le joueur a le droit d'ecriture via le tag [Command] qui demande au server de faire des choses
- `isClient` verifier.   

Comme on sait qu il vient de se connecter si il est un prefab de joueur et `isClient` et `isOwned`:
C est notre jouer local. 

On peut donc le stocker dans un singleton pour y faire appelle depuis d autres script "non reseau"
[<img width="1183" height="152" alt="image" src="https://github.com/user-attachments/assets/3babdf51-13f4-49ec-b851-1c155e230bc5" />](https://youtu.be/oKf_EAU-ct4?t=217)
https://youtu.be/oKf_EAU-ct4?t=217

Si on est le joueur local, on peut charger notre cle priver  et cle public ou la generer.
[<img width="674" height="245" alt="image" src="https://github.com/user-attachments/assets/ed625224-f76d-41e4-b611-1dd1cfefb673" />](https://youtu.be/oKf_EAU-ct4?t=238)
https://youtu.be/oKf_EAU-ct4?t=238

Cle priver, cest le truc qu il faut par partager car cest la preuve de qui vous etes.
Cle public, cest une emprunte qui permet de dire publiquement, "Si vous me le demander je peux prouver que je suis cette personne."


[<img width="1077" height="486" alt="image" src="https://github.com/user-attachments/assets/cf65c197-1d91-4758-8423-7c4a97e64e1d" />](https://youtu.be/oKf_EAU-ct4?t=273)
https://youtu.be/oKf_EAU-ct4?t=273

On av aller dire au server: "Hey gas, je suis `ldjaf...lkdjaf`"
Quand un client parle au server il utiliser la methode CmdBlablabla() avec un attribute `[Command]`.


Le server recoit l instruction d'executer le code suivant.
[<img width="1037" height="208" alt="image" src="https://github.com/user-attachments/assets/e81c244e-07e7-440d-9376-e102a7644cb7" />](https://youtu.be/oKf_EAU-ct4?t=267)  
https://youtu.be/oKf_EAU-ct4?t=267   
Sur le prefab du joueur en question, il generaire et stock un text unique et aleatoire un GUID.
Comme la crypographie ca utiliser du binaire, on le transforme en byte. depuis le format UTF8.

Et on envoie ce text que seuelement le server et bientot le client connaitrons.

Cela en utilisant un `[TargetRPC]`


Petit note:
- `[ClientRPC]` Envoie le message a tout le monde sur cette objet du reseaux
- `[TargetRPC]` Envoie le message a le proprietaire qui par le avec moi uniquement.


Du server a notre joueur uniquement:
[<img width="1330" height="502" alt="image" src="https://github.com/user-attachments/assets/d4692a8e-5087-41ca-808e-a3eb59dd7085" />](https://youtu.be/oKf_EAU-ct4?t=355)
https://youtu.be/oKf_EAU-ct4?t=355


Le serveur demande a notre joueur de signer les messages donner pour prover qu il possede la cle priver dont il parlait.
On retourne le binaire en text pour que ce soit humainement plus lisible.

Et on Sign le message binaire avec une cryptographie RSA.
``` cs
        public static  void SignDataWithRsaXmlPrivateKey(
            in byte[] bytesDataToSign, 
            in string privateKeyAsXmlFormat,
            out byte[] constructSignatureInBytes)
        {
            constructSignatureInBytes = null;
            using (RSA rsa = RSA.Create())
            {
                rsa.KeySize = KEY_SIZE;
                rsa.FromXmlString(privateKeyAsXmlFormat);
                constructSignatureInBytes= rsa.SignData(bytesDataToSign, HashAlgorithmName.SHA256, RSASignaturePadding.Pkcs1);
            }
        }
```

Cela permettra a n importe qui de savoir que la cle est signer par votre cle priver et il nous suffirat de comparer la cle public plustard.

Pour pas se prendre la tete on peu mettre le code de signateru dans un script static utilitaire.
Ici KeyPairRsaHolder.

Une fois signer, on recupere un tableau de byte qui possede le texte et la signature (un operation mathematique)

C est donc le moment de retourne ce joli message pour prouver au serveur que lon est qui on dit que lon est.

Client dit au server: Regarde c est moi ;)
[<img width="1433" height="442" alt="image" src="https://github.com/user-attachments/assets/b4179ef5-690a-4878-901c-5eadfdfd7e65" />](https://youtu.be/oKf_EAU-ct4?t=448)
https://youtu.be/oKf_EAU-ct4?t=448

Dans le code executer sur le server et appeller par le client, lon revoir le message a verifier.
[<img width="913" height="70" alt="image" src="https://github.com/user-attachments/assets/21e388b6-0d8d-4897-87f9-4f0bbbefc2a6" />
](https://youtu.be/oKf_EAU-ct4?t=523)
https://youtu.be/oKf_EAU-ct4?t=523
On lui donne a verifier:
- le message de base pour verifier que c est ce que on lui a demander
- le message pretenduement signer
- la cle publique pretendue du joueur

Si ca match, on est bon, sinon on kick|ban|ignore

```cs
public static bool VerifySignatureFromRawBytes(in byte[] dataUtf8, in byte[] signatureAsBytes, in string publicKeyXml)
        {
            using (RSA rsa = RSA.Create())
            {
                rsa.KeySize = KEY_SIZE;
                rsa.FromXmlString(publicKeyXml);
                return rsa.VerifyData(dataUtf8, signatureAsBytes, HashAlgorithmName.SHA256, RSASignaturePadding.Pkcs1);
            }
        }
```

Histoire de ne pas reutiliser `[ClientRPC]` ou TargetRPC]`, utilisons un `[SyncVar]`.

Ici pour communiquer fierement a tout les joueurs du jeux que ce client est belle et bien connecte et identifier dans la partie.
Personne ne connait les detailles, mais sont au courant que il est valide pour la cle publique donner.

[<img width="642" height="148" alt="image" src="https://github.com/user-attachments/assets/6ebcba75-8996-49a4-9a4f-0cd8cecddd86" />
](https://youtu.be/oKf_EAU-ct4?t=598)
https://youtu.be/oKf_EAU-ct4?t=598

Le Hook lui permet de dire: Si le server change cette valeur (TOUJOUR DEPUIS LE SERVER SEULEMENT) alors une fois chez le client on veut executer un bout de code.
Une sorte de listener pour le cient qui veut etre mis au courant de changement.

Ce qui nous permet ici de dire via un UnityEvent: "Hey les bas, il est valide c est bon".
Comme la valeur public avec deja ete changer au debut de la conversation on peut la relayer.
[<img width="910" height="266" alt="image" src="https://github.com/user-attachments/assets/b1226571-2ed4-4245-a6b3-e52af754f141" />
](https://youtu.be/oKf_EAU-ct4?t=641)
https://youtu.be/oKf_EAU-ct4?t=641

Bon comme notre joueur il est valide et identifier et que on aimerait sur des scripts non Mirror savoir les joueurs presents dans la partie.
On peut l ajouter un singleton qui garde en liste les joueurs valide et identifier disponible dans l application.

















