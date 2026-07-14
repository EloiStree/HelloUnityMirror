[<img width="1322" height="836" alt="image" src="https://github.com/user-attachments/assets/1d551403-d7ff-4734-9d15-0cd673e44986" />
](https://youtu.be/oKf_EAU-ct4?t=2)
https://youtu.be/oKf_EAU-ct4?t=2


Les commandes :
[<img width="759" height="209" alt="image" src="https://github.com/user-attachments/assets/f05f2623-32ce-4af4-881a-5fb6a30f3cc3" />](https://youtu.be/oKf_EAU-ct4?t=32
)   
https://youtu.be/oKf_EAU-ct4?t=32  


Il nous faut une clé cryptographique publique pour identifier les joueurs :
[<img width="878" height="212" alt="image" src="https://github.com/user-attachments/assets/88d6d9e5-25a9-4664-a59b-305f49c49b9f" />](https://youtu.be/oKf_EAU-ct4?t=97
)
https://youtu.be/oKf_EAU-ct4?t=97


Il nous faut un Network Identity sur le joueur pour faire partie du réseau Mirror :
[<img width="882" height="155" alt="image" src="https://github.com/user-attachments/assets/bd833004-0b48-43fa-a321-54c0f078da95" />](https://youtu.be/oKf_EAU-ct4?t=113)
https://youtu.be/oKf_EAU-ct4?t=113   



OnStartClient, utilisable si vous héritez de NetworkBehaviour, permet de dire quand le joueur est dans la partie officiellement.
[<img width="673" height="259" alt="image" src="https://github.com/user-attachments/assets/8b7aadd3-2910-40d4-bc61-0f9797fc5df4" />](https://youtu.be/oKf_EAU-ct4?t=142)
https://youtu.be/oKf_EAU-ct4?t=142   

<img width="615" height="257" alt="image" src="https://github.com/user-attachments/assets/ca8027cf-4994-4f6e-9c55-834034c03eba" />
- `isServer` dit si Unity est actuellement sur le serveur. (Attention, cela s'applique à tous les joueurs)
- `isOwned` dit si le joueur a le droit d'écriture via le tag [Command] qui demande au serveur de faire des choses.
- `isClient` permet de vérifier.   

Comme on sait qu'il vient de se connecter s'il est un prefab de joueur et que `isClient` et `isOwned` sont vrais :
C'est notre joueur local. 

On peut donc le stocker dans un singleton pour y faire appel depuis d'autres scripts « non réseau ».
[<img width="1183" height="152" alt="image" src="https://github.com/user-attachments/assets/3babdf51-13f4-49ec-b851-1c155e230bc5" />](https://youtu.be/oKf_EAU-ct4?t=217)
https://youtu.be/oKf_EAU-ct4?t=217

Si on est le joueur local, on peut charger notre clé privée et clé publique, ou la générer.
[<img width="674" height="245" alt="image" src="https://github.com/user-attachments/assets/ed625224-f76d-41e4-b611-1dd1cfefb673" />](https://youtu.be/oKf_EAU-ct4?t=238)
https://youtu.be/oKf_EAU-ct4?t=238

La clé privée, c'est le truc qu'il ne faut pas partager car c'est la preuve de qui vous êtes.
La clé publique, c'est une empreinte qui permet de dire publiquement : « Si vous me le demandez, je peux prouver que je suis cette personne. »


[<img width="1077" height="486" alt="image" src="https://github.com/user-attachments/assets/cf65c197-1d91-4758-8423-7c4a97e64e1d" />](https://youtu.be/oKf_EAU-ct4?t=273)
https://youtu.be/oKf_EAU-ct4?t=273

On va aller dire au serveur : « Hey les gars, je suis `ldjaf...lkdjaf` ».
Quand un client parle au serveur, il utilise la méthode CmdBlablabla() avec un attribut `[Command]`.


Le serveur reçoit l'instruction d'exécuter le code suivant.
[<img width="1037" height="208" alt="image" src="https://github.com/user-attachments/assets/e81c244e-07e7-440d-9376-e102a7644cb7" />](https://youtu.be/oKf_EAU-ct4?t=267)  
https://youtu.be/oKf_EAU-ct4?t=267   
Sur le prefab du joueur en question, il génère et stocke un texte unique et aléatoire : un GUID.
Comme la cryptographie utilise du binaire, on le transforme en bytes depuis le format UTF-8.

Et on envoie ce texte que seuls le serveur et bientôt le client connaîtront.

Cela en utilisant un `[TargetRPC]`.


Petite note :
- `[ClientRPC]` : Envoie le message à tout le monde sur cet objet du réseau.
- `[TargetRPC]` : Envoie le message au propriétaire qui parle avec moi uniquement.


Du serveur à notre joueur uniquement :
[<img width="1330" height="502" alt="image" src="https://github.com/user-attachments/assets/d4692a8e-5087-41ca-808e-a3eb59dd7085" />](https://youtu.be/oKf_EAU-ct4?t=355)
https://youtu.be/oKf_EAU-ct4?t=355


Le serveur demande à notre joueur de signer le message donné pour prouver qu'il possède la clé privée dont il parlait.
On retourne le binaire en texte pour que ce soit humainement plus lisible.

Et on signe le message binaire avec une cryptographie RSA.
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

Cela permettra à n'importe qui de savoir que c'est signé par votre clé privée et il nous suffira de comparer la clé publique plus tard.

Pour ne pas se prendre la tête, on peut mettre le code de signature dans un script utilitaire statique.
Ici KeyPairRsaHolder.

Une fois signé, on récupère un tableau de bytes qui possède le texte et la signature (une opération mathématique).

C'est donc le moment de retourner ce joli message pour prouver au serveur que l'on est qui l'on dit que l'on est.

Le client dit au serveur : « Regarde, c'est moi ;) »
[<img width="1433" height="442" alt="image" src="https://github.com/user-attachments/assets/b4179ef5-690a-4878-901c-5eadfdfd7e65" />](https://youtu.be/oKf_EAU-ct4?t=448)
https://youtu.be/oKf_EAU-ct4?t=448

Dans le code exécuté sur le serveur et appelé par le client, l'on reçoit le message à vérifier.
[<img width="913" height="70" alt="image" src="https://github.com/user-attachments/assets/21e388b6-0d8d-4897-87f9-4f0bbbefc2a6" />
](https://youtu.be/oKf_EAU-ct4?t=523)
https://youtu.be/oKf_EAU-ct4?t=523
On lui donne à vérifier :
- le message de base pour vérifier que c'est ce qu'on lui a demandé.
- le message prétendument signé.
- la clé publique prétendue du joueur.

Si ça matche, on est bon, sinon on kick|ban|ignore.

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

Histoire de ne pas réutiliser `[ClientRPC]` ou `[TargetRPC]`, utilisons un `[SyncVar]`.

Ici, pour communiquer fièrement à tous les joueurs du jeu que ce client est bel et bien connecté et identifié dans la partie.
Personne ne connaît les détails, mais ils sont au courant qu'il est valide pour la clé publique donnée.

[<img width="642" height="148" alt="image" src="https://github.com/user-attachments/assets/6ebcba75-8996-49a4-9a4f-0cd8cecddd86" />
](https://youtu.be/oKf_EAU-ct4?t=598)
https://youtu.be/oKf_EAU-ct4?t=598

Le Hook lui permet de dire : si le serveur change cette valeur (TOUJOURS DEPUIS LE SERVEUR SEULEMENT), alors une fois chez le client, on veut exécuter un bout de code.
Une sorte de listener pour le client qui veut être mis au courant des changements.

Ce qui nous permet ici de dire via un UnityEvent : « Hey en bas, il est valide, c'est bon ».
Comme la valeur publique a déjà été changée au début de la conversation, on peut la relayer.
[<img width="910" height="266" alt="image" src="https://github.com/user-attachments/assets/b1226571-2ed4-4245-a6b3-e52af754f141" />
](https://youtu.be/oKf_EAU-ct4?t=641)
https://youtu.be/oKf_EAU-ct4?t=641

Bon, comme notre joueur est valide et identifié et qu'on aimerait, sur des scripts non Mirror, savoir les joueurs présents dans la partie,
on peut l'ajouter à un singleton qui garde en liste les joueurs valides et identifiés disponibles dans l'application.


On peut donc constater dans l'éditeur que le joueur a bien chargé sa clé privée et signé le message. 
[<img width="900" height="273" alt="image" src="https://github.com/user-attachments/assets/4c0c191c-0085-494c-9141-4c42b73ad5a1" />](https://youtu.be/oKf_EAU-ct4?t=681)
https://youtu.be/oKf_EAU-ct4?t=681


On peut préparer deux ou trois champs pour aider.
[<img width="879" height="242" alt="image" src="https://github.com/user-attachments/assets/36d0a74d-da97-42eb-8c33-eeb34d902ea9" />](https://youtu.be/oKf_EAU-ct4?t=691)
https://youtu.be/oKf_EAU-ct4?t=691   

Player Network Id, c'est un entier donné par Mirror à chaque joueur.
Public Key, c'est notre identifiant vérifié.
IsOwned, c'est pour bien rappeler que l'on est sur le joueur qui possède le prefab instancié par Mirror.
Et le UnityEvent est là pour le GameDesigner qui peut utiliser la clé.
_Par exemple pour créer un Blocky (une représentation graphique d'une clé publique)._
Exemple : https://youtu.be/oKf_EAU-ct4?t=721


En passant, voilà notre debug pour voir (quand on est sur le serveur) l'état de la vérification :
<img width="853" height="214" alt="image" src="https://github.com/user-attachments/assets/564adc58-3787-4317-8c5b-2585a8ffe1cf" />



----------

Vous pouvez vous faciliter la vie en créant des petits scripts gérant IfOwnedEvent.
[<img width="885" height="382" alt="image" src="https://github.com/user-attachments/assets/e4f11398-3e16-49cd-ae78-0f0b1cc6d0be" />](https://youtu.be/oKf_EAU-ct4?t=857)
https://youtu.be/oKf_EAU-ct4?t=857  


Envoyer et recevoir des textes et des bytes comme une sorte de WebSocket serveur ;)
[<img width="887" height="237" alt="image" src="https://github.com/user-attachments/assets/6c1eee2d-95ba-4d28-b2a1-f4b7194dc51b" />
](https://youtu.be/oKf_EAU-ct4?t=940)


---------------------


Si vous faites un jeu en réseau, il est important de compresser les données, pour la bande passante et pour votre portefeuille.
Car chaque byte qui passe sur le réseau, c'est vous qui le payez.

Voici par exemple comment compresser deux joysticks de la manette sur 32 bits (un entier) :
[<img width="1397" height="297" alt="image" src="https://github.com/user-attachments/assets/641b6b90-c505-417c-a474-c506596479f2" />](https://youtu.be/oKf_EAU-ct4?t=1090)
https://youtu.be/oKf_EAU-ct4?t=1090

Si ça vous intéresse.




-----------

Et donc voilà, si vous voulez un récapitulatif :
[<img width="1607" height="833" alt="image" src="https://github.com/user-attachments/assets/a9b903d8-78b3-4333-8e14-9471f65bef43" />](https://youtu.be/oKf_EAU-ct4?t=1214)
https://youtu.be/oKf_EAU-ct4?t=1214

Code Unity et résumé :
https://github.com/EloiStree/2024_06_01_unity_hello_mirror_drone_soccer

Code source du code de la vidéo :
https://github.com/EloiStree/2024_06_01_unity_hello_mirror_drone_soccer/blob/4ce98d2287e10f9586c9b81c436c231bad8fd5ad/Assets/00__DroneSoccerXR/Unstore%20Code/Runtime/Mirror/MirrorMono_RsaKeyIdentity.cs



---------


Vous pouvez trouver toute une série de scripts intéressants ici que j'avais faits à l'époque :
[<img width="1410" height="977" alt="image" src="https://github.com/user-attachments/assets/6aa6d37a-6122-49d2-a038-71f036f566cb" />](https://github.com/EloiStree/2024_06_01_unity_hello_mirror_drone_soccer/tree/4ce98d2287e10f9586c9b81c436c231bad8fd5ad/Assets/00__DroneSoccerXR/Unstore%20Code/Runtime)  
https://github.com/EloiStree/2024_06_01_unity_hello_mirror_drone_soccer/tree/4ce98d2287e10f9586c9b81c436c231bad8fd5ad/Assets/00__DroneSoccerXR/Unstore%20Code/Runtime  
