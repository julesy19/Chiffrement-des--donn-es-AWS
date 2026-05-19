# Chiffrement des donnes AWS
# Atelier guidé : chiffrement des données au repos à l'aide des options de chiffrement AWS

Présentation de l'atelier et des objectifs
Dans cet atelier, vous allez passer en revue le chiffrement des données par défaut et le chiffrement AWS Key Management Service (AMS KMS) utilisés pour chiffrer des données au repos. Vous allez passer en revue le chiffrement par défaut des objets stockés dans Amazon Simple Storage Service (Amazon S3). Vous allez créer une clé AWS KMS et l'utiliser pour chiffrer des objets stockés dans des volumes Amazone Elastic Block Store (Amazon EBS). Vous allez également observer comment AWS CloudTrail fournit un journal d'audit de l'utilisation de la clé AWS KMS et comment la désactivation de cette clé affecte l'accès aux données. 

À la fin de cet atelier, vous serez en mesure d'effectuer les tâches suivantes :
passer en revue le chiffrement par défaut fourni par Amazon S3 ;
accéder à l'objet Amazon S3 chiffré ;
créer une clé AWS KMS gérée par le client pour chiffrer et déchiffrer des données au repos ;
créer et attacher un volume de données chiffré à une instance EC2 existante ;
désactiver et réactiver une clé AWS KMS, et observer les effets sur l'accès aux données ;
surveiller l'utilisation de la clé de chiffrement à l'aide de l'historique des événements CloudTrail ;
passer en revue la rotation des clés.

Dans cet atelier, vous allez commencer avec les composants architecturaux suivants : 
un compartiment S3 pour stocker des images ; 
une instance EC2 ; 
une clé AWS KMS à utiliser pour le chiffrement pendant l'atelier.
Le schéma suivant illustre ces composants :

<img width="302" height="271" alt="image" src="https://github.com/user-attachments/assets/9a69791f-fc88-40b7-b545-12086d4fe7bd" />



<----------------------------------------->


À la fin de cet atelier, vous aurez créé l'architecture présentée dans le schéma suivant. Ce schéma montre certaines des actions que vous allez effectuer et les communications ayant lieu entre Amazon EC2, Amazon EBS, AWS KMS et CloudTrail pendant l'atelier.



<img width="477" height="461" alt="image" src="https://github.com/user-attachments/assets/1127506f-c048-4352-920a-2e67aedd12e4" />


Description du schéma : un utilisateur est connecté aux ressources cloud AWS via Internet. Le compartiment S3 nommé ImageBucket contient un objet chiffré côté serveur et des clés gérées par Amazon S3 (SSE-S3). Le volume chiffré est attaché à l'instance EC2 nommée LabInstance. Des demandes de chiffrement ou de déchiffrement des volumes EBS sont envoyées à AWS KMS, où la clé gérée par le client nommée MyKMSKey est stockée. CloudTrail consigne l'utilisation de la clé AWS KMS sous forme d'événements.


<img width="641" height="176" alt="image" src="https://github.com/user-attachments/assets/9ddfa6b6-f4c0-4ffa-a75f-acf1f21aac85" />


# Tâche 1 : revue du chiffrement par défaut des objets d'un compartiment S3
Tâche 1 : revue du chiffrement par défaut des objets d'un compartiment S3
Au cours de cette tâche, vous allez charger un fichier image dans un compartiment S3 et passer en revue le chiffrement par défaut fourni par Amazon S3. 

Téléchargez le fichier clock.png sur votre ordinateur.
Ensuite, vous allez localiser le compartiment S3 et analyser ses paramètres de chiffrement.
Dans la barre de recherche en haut de la console de gestion AWS, recherchez et choisissez S3 pour ouvrir la console Amazon S3.
Dans le volet de navigation, choisissez Compartiments.
Sélectionnez le lien correspondant au compartiment dont le nom contient imagebucket.
Choisissez l'onglet Propriétés.
Dans la section Chiffrement par défaut, notez le paramètre Chiffrement côté serveur est automatiquement appliqué aux nouveaux objets stockés dans ce compartiment.
Remarque : Amazon S3 applique désormais le chiffrement côté serveur avec des clés gérées par Amazon S3 (SSE-S3) comme chiffrement de base pour chaque compartiment Amazon S3.
Ensuite, vous allez charger un fichier dans le compartiment.
En haut de la page, sélectionnez l'onglet Objets.
Choisissez Charger.
Choisissez Ajouter des fichiers.
Recherchez le fichier clock.png que vous avez téléchargé sur votre ordinateur et sélectionnez-le.
Développez l'option Autorisations.
Choisissez Accorder l'accès en lecture publique.
Le message d'avertissement suivant s'affiche : Granting public-read access is not recommended (Il n'est pas recommandé d'accorder un accès en lecture publique).
Pour confirmer que vous avez pris connaissance de l'avertissement, sélectionnez I understand the risk of granting public-read access to the specified objects (Je comprends les risques liés à l'octroi d'un accès en lecture publique aux objets spécifiés).
En bas de la page, choisissez Charger.

Une fois le chargement terminé, dans le coin supérieur droit, choisissez Fermer.
Le fichier clock.png est désormais répertorié en tant qu'objet dans le compartiment.
Pour ouvrir les propriétés du fichier, choisissez le fichier clock.png.
Dans la section Paramètres de chiffrement côté serveur, notez les paramètres suivants : 
Ce paramètre de chiffrement est activé, et le message suivant s'affiche : Server-side encryption protects data at rest (Le chiffrement côté serveur protège les données au repos).
Pour Type de chiffrement, le message suivant s'affiche : Server-side encryption with Amazon S3 managed keys (SSE-S3)(Chiffrement côté serveur avec clés gérées par Amazon S3 [SSE-S3]).
Pour ouvrir l'image, dans la section Présentation de l'objet, choisissez l'URL de l'objet. 
Vous pouvez utiliser cette méthode pour accéder à un objet Amazon S3 à partir de l'Internet public.
 L'image clock.png s'ouvre dans un onglet du navigateur. 
 Remarque : l'objet est chiffré. Il doit être déchiffré pour s'afficher. Le service gère ce déchiffrement de manière transparente.

Au cours de cette tâche, vous avez passé en revue les paramètres de chiffrement du compartiment. Vous avez ensuite chargé un objet dans le compartiment et vous y avez accédé à l'aide d'un lien public associé à l'objet. Amazon S3 a déchiffré l'objet de manière transparente avant de l'afficher dans le navigateur.


<----------------------->



<img width="916" height="238" alt="image" src="https://github.com/user-attachments/assets/749eba3e-71f5-4c88-b58a-e94953f2280f" />

<----------------------->


<img width="663" height="449" alt="image" src="https://github.com/user-attachments/assets/5b458929-6f39-4fae-bcee-731895662ccd" />




<img width="517" height="347" alt="image" src="https://github.com/user-attachments/assets/f6238ed9-8e68-45d1-8325-ffb2a9919726" />


<--------------------->

<img width="949" height="427" alt="image" src="https://github.com/user-attachments/assets/941ad77e-83d4-48cb-9899-baca1201db6f" />


<-------------------->




# Tâche 2 : création d'une clé AWS KMS.
Au cours de cette tâche, vous allez créer une clé AWS KMS gérée par le client. Plus tard dans l'atelier, vous utiliserez la clé AWS KMS que vous avez créée pour générer, chiffrer et déchiffrer des clés de données. Les clés de données seront partagées avec Amazon EC2. Les clés de données servent à chiffrer des données réelles stockées sur des volumes EBS.

Dans la barre de recherche de la console de gestion AWS, recherchez KMS et choisissez Key Management Service pour ouvrir la console AWS KMS.

Dans le volet de navigation, choisissez Clés gérées par le client.

   Conseil : pour ouvrir le volet de navigation, choisissez l'icône de menu  dans le coin supérieur gauche. 

Choisissez Créer une clé.

Pour Type de clé, choisissez Symétrique.

   Remarque : les clés symétriques ne quittent jamais AWS KMS sans chiffrement. La clé que vous créez sera une clé secrète de 256 bits.

Choisissez Suivant.  

Pour Alias, saisissez MyKMSKey.

Choisissez Suivant. 

Sur la page Définir des autorisations d'administration de clé, pour Administrateurs de clé, recherchez le rôle voclabs et sélectionnez-le.

   Remarque : Les administrateurs de clé sont des utilisateurs ou des rôles qui gèrent l'accès à la clé de chiffrement. Vous êtes actuellement connecté à la console avec le rôle voclabs. Ce rôle d'utilisateur est affiché dans le coin supérieur droit de la console, à gauche de la région AWS. 

Choisissez Suivant.

Sur la page Définir des autorisations d'utilisation de clé, pour Utilisateurs de clé, recherchez le rôle voclabs et sélectionnez-le de nouveau.

   Remarque : les utilisateurs de clé sont les utilisateurs ou les rôles qui utilisent la clé pour chiffrer et déchiffrer les données. 

Choisissez Suivant.

En bas de la page Vérification, choisissez Terminer.

  Votre clé s'affiche dans la liste des clés gérées par le client.
 Remarque : si vous avez essayé cet atelier au cours des 7 derniers jours, il 


<img width="951" height="198" alt="image" src="https://github.com/user-attachments/assets/980d2b60-7c49-40eb-8535-810b956e4ed5" />



<----------------------->


# Tâche 3 : création et attachement d'un volume de données chiffré à une instance EC2

Au cours de cette tâche, vous allez créer un volume EBS chiffré à l'aide de la clé KMS que vous avez créée lors de la tâche précédente et l'attacher à votre instance EC2. Lorsque vous attachez le volume chiffré, l'instance EC2 extrait la clé de données d'AWS KMS et l'utilise pour déchiffrer les données du volume EBS. Lors des tâches suivantes, vous allez examiner l'historique des événements CloudTrail pour observer les appels passés au service AWS KMS.

Tout d'abord, vous allez observer le volume racine.

<img width="838" height="308" alt="image" src="https://github.com/user-attachments/assets/1838c20d-1d0f-4a48-9490-688688872097" />


<------------------------>


<img width="830" height="164" alt="image" src="https://github.com/user-attachments/assets/8233e0be-e6a0-40b9-8243-933d4436ffd9" />

<----------------------->

Tâche 4 : désactivation de la clé de chiffrement et observation des effets
Au cours de cette tâche, vous allez désactiver temporairement la clé AWS KMS que vous avez précédemment utilisée pour chiffrer le volume EBS. Vous allez ensuite observer les effets de la désactivation de la clé sur l'accès aux données chiffrées.

Tout d'abord, vous allez désactiver la clé AWS KMS que vous avez utilisée pour chiffrer le volume EBS.


<img width="870" height="346" alt="image" src="https://github.com/user-attachments/assets/a19e43ff-1f60-4245-9384-79f0610800c2" />


<---------------------->


<img width="814" height="241" alt="image" src="https://github.com/user-attachments/assets/3bcff7ce-a98e-4ac4-bd8f-ed5168b8f39b" />


<-------------------->
Accédez à la console Amazon EC2.

Dans le volet de navigation, sélectionnez Volumes.
Sélectionnez le lien du volume que vous avez créé.
Choisissez Actions > Détacher un volume.
Dans la fenêtre Détacher qui s'affiche, choisissez Détacher.


<img width="832" height="226" alt="image" src="https://github.com/user-attachments/assets/7c263cbc-2f9b-4820-9d94-7d65e8ed92b3" />


<-------------------->


En regard d'Actions, cliquez sur le bouton Actualiser pour actualiser la page.
Choisissez Actions > Attacher un volume.
Pour Instance, choisissez votre instance dans la liste déroulante.
Choisissez Attacher un volume.
 Un message semblable au suivant s'affiche : Volume vol-072d7cf3173a13151 cannot be attached. The encrypted volume was unable to access the KMS key (Impossible d'attacher le volume vol-072d7cf3173a13151). Le volume chiffré n'a pas pu accéder à la clé KMS).

 
<img width="896" height="354" alt="image" src="https://github.com/user-attachments/assets/0e615308-b645-4f4f-bfc8-a3f731852149" />


L'opération échoue comme prévu, car la clé AWS KMS utilisée pour chiffrer le volume de données est désormais désactivée et ne peut plus être utilisée pour déchiffrer les données.




Dans le volet de navigation, sélectionnez Historique des événements.
Conseil : pour ouvrir le volet de navigation, choisissez l'icône de menu  dans le coin supérieur gauche.
Remarque : CloudTrail fournit un journal d'audit des appels d'API effectués sur le compte AWS. L'historique des événements permet de consulter les événements des 90 derniers jours d'activité du compte.


Les événements récents sont affichés.
Sélectionnez le lien correspondant à l'événement DisableKey.
Dans la section Enregistrement d'événement, passez en revue les détails de l'événement. Vous devriez voir des informations semblables à l'enregistrement suivant :


Enregistrement d'événement

Vue JSON
{
    "eventVersion": "1.11",
    "userIdentity": {
        "type": "AssumedRole",
        "principalId": "AROA2NCBAL3RPBAGXM7CZ:user4632195=Mamadou__Sy",
        "arn": "arn:aws:sts::715248393954:assumed-role/voclabs/user4632195=Mamadou__Sy",
        "accountId": "715248393954",
        "accessKeyId": "ASIA2NCBAL3RMAXUTZQK",
        "sessionContext": {
            "sessionIssuer": {
                "type": "Role",
                "principalId": "AROA2NCBAL3RPBAGXM7CZ",
                "arn": "arn:aws:iam::715248393954:role/voclabs",
                "accountId": "715248393954",
                "userName": "voclabs"
            },
            "attributes": {
                "creationDate": "2026-05-18T04:14:33Z",
                "mfaAuthenticated": "false"
            }
        }
    },
    "eventTime": "2026-05-18T05:41:19Z",
    "eventSource": "kms.amazonaws.com",
    "eventName": "DisableKey",
    "awsRegion": "us-east-1",
    "sourceIPAddress": "174.89.99.59",
    "userAgent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/148.0.0.0 Safari/537.36",
    "requestParameters": {
        "keyId": "55487ec5-4c6b-4c87-80e5-3cfa8bcfc271"
    },
    "responseElements": {
        "keyId": "arn:aws:kms:us-east-1:715248393954:key/55487ec5-4c6b-4c87-80e5-3cfa8bcfc271"
    },
    "requestID": "8dbc93c3-34e9-41fc-9bb4-72d84ab8f4c2",
    "eventID": "2f71316e-829b-4816-9202-797c00ffdf53",
    "readOnly": false,
    "resources": [
        {
            "accountId": "715248393954",
            "type": "AWS::KMS::Key",
            "ARN": "arn:aws:kms:us-east-1:715248393954:key/55487ec5-4c6b-4c87-80e5-3cfa8bcfc271"
        }
    ],
    "eventType": "AwsApiCall",
    "managementEvent": true,
    "recipientAccountId": "715248393954",
    "eventCategory": "Management",
    "tlsDetails": {
        "tlsVersion": "TLSv1.3",
        "cipherSuite": "TLS_AES_256_GCM_SHA384",
        "clientProvidedHostHeader": "kms.us-east-1.amazonaws.com",
        "keyExchange": "X25519MLKEM768"
    },
    "sessionCredentialFromConsole": "true"
}

Dans la section Détails, examinez les détails de l'événement. L'heure de l'événement indique que vous avez désactivé la clé AWS KMS il y a quelques minutes.
Vous allez ensuite examiner l'événement AttachVolume, qui s'est produit juste après l'événement DisableKey.

<------------------------------>


Dans le volet de navigation, sélectionnez Historique des événements.
Sélectionnez le lien de l'événement AttachVolume.
Dans la section Enregistrement d'événement, passez en revue les détails de l'événement. Vous devriez voir des informations semblables à l'enregistrement suivant 


<------------------------------>

aTTACHvOLUME

{
    "eventVersion": "1.11",
    "userIdentity": {
        "type": "AssumedRole",
        "principalId": "AROA2NCBAL3RPBAGXM7CZ:user4632195=Mamadou__Sy",
        "arn": "arn:aws:sts::715248393954:assumed-role/voclabs/user4632195=Mamadou__Sy",
        "accountId": "715248393954",
        "accessKeyId": "ASIA2NCBAL3RBYRBYMJV",
        "sessionContext": {
            "sessionIssuer": {
                "type": "Role",
                "principalId": "AROA2NCBAL3RPBAGXM7CZ",
                "arn": "arn:aws:iam::715248393954:role/voclabs",
                "accountId": "715248393954",
                "userName": "voclabs"
            },
            "attributes": {
                "creationDate": "2026-05-18T04:14:33Z",
                "mfaAuthenticated": "false"
            }
        }
    },
    "eventTime": "2026-05-18T05:47:50Z",
    "eventSource": "ec2.amazonaws.com",
    "eventName": "AttachVolume",
    "awsRegion": "us-east-1",
    "sourceIPAddress": "174.89.99.59",
    "userAgent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/148.0.0.0 Safari/537.36",
    "errorCode": "Client.CustomerKeyHasBeenRevoked",
    "errorMessage": "Volume vol-0894f908458cca170 cannot be attached. The encrypted volume was unable to access the KMS key.",
    "requestParameters": {
        "volumeId": "vol-0894f908458cca170",
        "instanceId": "i-0c9d812c0cfa2790f",
        "device": "/dev/sdf",
        "deleteOnTermination": false
    },
    "responseElements": null,
    "requestID": "00c992ec-cb32-4371-9edd-48caa9eb28e5",
    "eventID": "435a9a86-8afe-41b7-a09a-931a25287d3f",
    "readOnly": false,
    "eventType": "AwsApiCall",
    "managementEvent": true,
    "recipientAccountId": "715248393954",
    "eventCategory": "Management",
    "tlsDetails": {
        "tlsVersion": "TLSv1.3",
        "cipherSuite": "TLS_AES_128_GCM_SHA256",
        "clientProvidedHostHeader": "ec2.us-east-1.amazonaws.com"
    },
    "sessionCredentialFromConsole": "true"
}




 Les détails indiquent que la demande d'attachement du volume à l'instance a échoué. 

  En effet, le processus d'attachement du volume a remarqué que le volume de données était chiffré avec la clé MyKMSKey. Amazon EC2 a donc contacté AWS KMS pour obtenir la clé de données brute pour déchiffrer le volume. AWS KMS a refusé la demande, car la clé AWS KMS utilisée pour chiffrer la clé de données avec laquelle le volume EBS a été chiffré a été désactivée.

Vous allez ensuite réactiver la clé AWS KMS et rattacher le volume EBS.
Pour réactiver MyKMSKey, retournez à la console AWS KMS.
Sélectionnez  MyKMSKey, puis choisissez Actions de clé > Activer.


<img width="812" height="170" alt="image" src="https://github.com/user-attachments/assets/2838138a-9da3-4049-9f31-554b7e0da090" />


<------------------------------>


Pour attacher de nouveau le volume, revenez à la console Amazon EC2.
Dans le volet de navigation, sélectionnez Volumes.
Sélectionnez votre volume, puis choisissez Actions > Attacher un volume.
Sur la page Attacher un volume, pour Instance, choisissez votre instance, puis Attacher un volume.
 Le volume a été attaché avec succès.

 <img width="833" height="182" alt="image" src="https://github.com/user-attachments/assets/31b95c70-01b8-4da5-8c70-e69dd5be0557" />



 # Tâche 5 : analyse de l'activité d'AWS KMS à l'aide de CloudTrail

Au cours de cette tâche, vous allez accéder à l'historique des événements CloudTrail pour rechercher les événements liés à vos opérations de chiffrement. La fonctionnalité de journal d'audit CloudTrail est une fonctionnalité de sécurité importante, et il est judicieux de surveiller la manière dont les clés AWS KMS sont utilisées dans votre compte.
Tout d'abord, vous allez accéder à l'historique des événements CloudTrail.


Accédez à la console CloudTrail. 

Dans le volet de navigation, sélectionnez Historique des événements.
Dans la liste Historique des événements, notez la colonne nommée Source de l'événement.
Chaque fois qu'un appel d'API à un service AWS se produit dans la région que vous avez sélectionnée, l'événement et le service AWS qui l'a signalé sont répertoriés si le service signale de tels événements à CloudTrail.
Ensuite, vous allez filtrer l'historique des événements pour afficher uniquement les événements signalés par le service AWS KMS. 
Sous Attributs de recherche, dans la liste déroulante Read-only (Lecture seule), choisissez Source de l'événement.
Dans la barre de recherche Enter an event source (Saisissez une source d'événement), saisissez kms, puis choisissez kms.amazonaws.com.
Vous allez ensuite analyser quelques événements CloudTrail clés.


<img width="646" height="358" alt="image" src="https://github.com/user-attachments/assets/b06956af-6130-4b6d-9689-fb5004b29437" />


<------------------------>


Enregistrement d'événement Infos
Copier
Vue JSON
{
    "eventVersion": "1.11",
    "userIdentity": {
        "type": "AssumedRole",
        "principalId": "AROA2NCBAL3RPBAGXM7CZ:user4632195=Mamadou__Sy",
        "arn": "arn:aws:sts::715248393954:assumed-role/voclabs/user4632195=Mamadou__Sy",
        "accountId": "715248393954",
        "accessKeyId": "ASIA2NCBAL3RMBTONGYQ",
        "sessionContext": {
            "sessionIssuer": {
                "type": "Role",
                "principalId": "AROA2NCBAL3RPBAGXM7CZ",
                "arn": "arn:aws:iam::715248393954:role/voclabs",
                "accountId": "715248393954",
                "userName": "voclabs"
            },
            "attributes": {
                "creationDate": "2026-05-18T04:14:33Z",
                "mfaAuthenticated": "false"
            }
        },
        "invokedBy": "ec2-frontend-api.amazonaws.com"
    },
    "eventTime": "2026-05-18T06:16:58Z",
    "eventSource": "kms.amazonaws.com",
    "eventName": "CreateGrant",
    "awsRegion": "us-east-1",
    "sourceIPAddress": "ec2-frontend-api.amazonaws.com",
    "userAgent": "ec2-frontend-api.amazonaws.com",
    "requestParameters": {
        "constraints": {
            "encryptionContextSubset": {
                "aws:ebs:id": "vol-0894f908458cca170"
            }
        },
        "retiringPrincipal": "ec2.us-east-1.amazonaws.com",
        "operations": [
            "Decrypt"
        ],
        "keyId": "arn:aws:kms:us-east-1:715248393954:key/55487ec5-4c6b-4c87-80e5-3cfa8bcfc271",
        "granteePrincipal": "arn:aws:sts::715248393954:assumed-role/aws:ec2-infrastructure/i-0c9d812c0cfa2790f"
    },
    "responseElements": {
        "grantId": "d0c79b0dd35cdb5284835fd1805d64afea998325a5cad1c6753f67a9d284fe3e",
        "keyId": "arn:aws:kms:us-east-1:715248393954:key/55487ec5-4c6b-4c87-80e5-3cfa8bcfc271"
    },
    "requestID": "b7fb8d8b-49f1-4f89-a200-7962685540ef",
    "eventID": "ccbed48e-00bd-3ccd-9b2b-b1a04add39f7",
    "readOnly": false,
    "resources": [
        {
            "accountId": "715248393954",
            "type": "AWS::KMS::Key",
            "ARN": "arn:aws:kms:us-east-1:715248393954:key/55487ec5-4c6b-4c87-80e5-3cfa8bcfc271"
        }
    ],
    "eventType": "AwsApiCall",
    "managementEvent": true,
    "recipientAccountId": "715248393954",
    "sharedEventID": "7941ede6-b598-4736-a076-5c19101b8670",
    "vpcEndpointId": "AWS Internal",
    "vpcEndpointAccountId": "AWS Internal",
    "eventCategory": "Management",
    "sessionCredentialFromConsole": "true"
}


<----------------------------->

Analyse : cet événement permet aux principaux AWS d'utiliser des clés AWS KMS dans le cadre d'opérations cryptographiques. L'instance EC2 envoie une requête CreateGrant pour pouvoir déchiffrer la clé de données.

Revenez à la liste Historique des événements et choisissez le lien de l'événement Decrypt. S'il y a plusieurs événements Decrypt, sélectionnez l'un d'eux.


<img width="808" height="285" alt="image" src="https://github.com/user-attachments/assets/b129afac-0329-4d63-98f8-b069eabdf0c3" />



Remarque : si l'événement Decrypt ne s'affiche pas, patientez quelques minutes.
Dans la section Enregistrement d'événement, passez en revue les détails de l'événement. Les détails ressemblent à l'image suivante. Sur cette image, les détails essentiels de l'enregistrement de l'événement sont surlignés.

Enregistrement d'événement Infos
Copier
Vue JSON
{
    "eventVersion": "1.11",
    "userIdentity": {
        "type": "AssumedRole",
        "principalId": "715248393954:aws:ec2-infrastructure:i-0c9d812c0cfa2790f",
        "arn": "arn:aws:sts::715248393954:assumed-role/aws:ec2-infrastructure/i-0c9d812c0cfa2790f",
        "accountId": "715248393954",
        "accessKeyId": "ASIA2NCBAL3RITJGADU7",
        "sessionContext": {
            "sessionIssuer": {
                "type": "Role",
                "principalId": "715248393954:aws:ec2-infrastructure",
                "arn": "arn:aws:iam::715248393954:role/aws:ec2-infrastructure",
                "accountId": "715248393954",
                "userName": "aws:ec2-infrastructure"
            },
            "attributes": {
                "creationDate": "2026-05-18T06:16:59Z",
                "mfaAuthenticated": "false"
            }
        },
        "invokedBy": "AWS Internal",
        "inScopeOf": {
            "issuerType": "AWS::EC2::Instance",
            "credentialsIssuedTo": "arn:aws:ec2:us-east-1:715248393954:instance/i-0c9d812c0cfa2790f"
        }
    },
    "eventTime": "2026-05-18T06:16:59Z",
    "eventSource": "kms.amazonaws.com",
    "eventName": "Decrypt",
    "awsRegion": "us-east-1",
    "sourceIPAddress": "AWS Internal",
    "userAgent": "AWS Internal",
    "requestParameters": {
        "encryptionContext": {
            "aws:ebs:id": "vol-0894f908458cca170"
        },
        "encryptionAlgorithm": "SYMMETRIC_DEFAULT"
    },
    "responseElements": null,
    "additionalEventData": {
        "keyMaterialId": "f9865a6a5bab38dcf11581758310b581a19385034dd450760181fdb7a7f9895e"
    },
    "requestID": "b7fb8d8b-49f1-4f89-a200-7962685540ef",
    "eventID": "910bcc02-9784-4292-ad7f-7a941cc9d8da",
    "readOnly": true,
    "resources": [
        {
            "accountId": "715248393954",
            "type": "AWS::KMS::Key",
            "ARN": "arn:aws:kms:us-east-1:715248393954:key/55487ec5-4c6b-4c87-80e5-3cfa8bcfc271"
        }
    ],
    "eventType": "AwsApiCall",
    "managementEvent": true,
    "recipientAccountId": "715248393954",
    "eventCategory": "Management"
}

Analyse : Decrypt fait suite à un événement CreateGrant réussi. L'enregistrement présente les détails de la clé AWS KMS nommée MyKMSKey que l'instance EC2 utilise pour déchiffrer la clé chiffrée, qui est ensuite utilisée pour déchiffrer les données du volume EBS.

  
<------------------------->

Revenez à la liste Historique des événements, choisissez les liens des événements suivants et observez les détails de ces derniers.
Pour GenerateDataKeyWithoutPlainText, un événement s'est produit lorsqu'une demande de génération de clé pour chiffrer les données du volume EBS a été envoyée à AWS KMS.
VOus pouvez également consulter les événements tels que RetireGrant.


<img width="501" height="342" alt="image" src="https://github.com/user-attachments/assets/35adf267-16b5-45b4-8d4f-f799a0d7911b" />


<------------------------>


Enregistrement d'événement Infos
Copier
Vue JSON
{
    "eventVersion": "1.11",
    "userIdentity": {
        "type": "AssumedRole",
        "principalId": "AROA2NCBAL3RPBAGXM7CZ:user4632195=Mamadou__Sy",
        "arn": "arn:aws:sts::715248393954:assumed-role/voclabs/user4632195=Mamadou__Sy",
        "accountId": "715248393954",
        "accessKeyId": "ASIA2NCBAL3RJEXXE3LI",
        "sessionContext": {
            "sessionIssuer": {
                "type": "Role",
                "principalId": "AROA2NCBAL3RPBAGXM7CZ",
                "arn": "arn:aws:iam::715248393954:role/voclabs",
                "accountId": "715248393954",
                "userName": "voclabs"
            },
            "attributes": {
                "creationDate": "2026-05-18T04:14:33Z",
                "mfaAuthenticated": "false"
            }
        },
        "invokedBy": "ec2-frontend-api.amazonaws.com"
    },
    "eventTime": "2026-05-18T05:25:19Z",
    "eventSource": "kms.amazonaws.com",
    "eventName": "GenerateDataKeyWithoutPlaintext",
    "awsRegion": "us-east-1",
    "sourceIPAddress": "ec2-frontend-api.amazonaws.com",
    "userAgent": "ec2-frontend-api.amazonaws.com",
    "requestParameters": {
        "encryptionContext": {
            "aws:ebs:id": "vol-0894f908458cca170"
        },
        "numberOfBytes": 64,
        "keyId": "arn:aws:kms:us-east-1:715248393954:key/55487ec5-4c6b-4c87-80e5-3cfa8bcfc271"
    },
    "responseElements": null,
    "additionalEventData": {
        "keyMaterialId": "f9865a6a5bab38dcf11581758310b581a19385034dd450760181fdb7a7f9895e"
    },
    "requestID": "328be315-fd9b-42ee-ad12-8b4f4b0ae25b",
    "eventID": "c5679cb7-5e88-3256-81fa-c86638fdb76c",
    "readOnly": true,
    "resources": [
        {
            "accountId": "715248393954",
            "type": "AWS::KMS::Key",
            "ARN": "arn:aws:kms:us-east-1:715248393954:key/55487ec5-4c6b-4c87-80e5-3cfa8bcfc271"
        }
    ],
    "eventType": "AwsApiCall",
    "managementEvent": true,
    "recipientAccountId": "715248393954",
    "sharedEventID": "ffe4c640-83d6-46ed-bb08-3526b3ea6d5d",
    "vpcEndpointId": "AWS Internal",
    "vpcEndpointAccountId": "AWS Internal",
    "eventCategory": "Management",
    "sessionCredentialFromConsole": "true"
}



<img width="841" height="335" alt="image" src="https://github.com/user-attachments/assets/f147d7c0-048e-4c2e-a42f-9b6ca29dc31e" />



CloudTrail enregistre toutes les activités de l'API AWS KMS. L'analyse de ces entrées du journal peut vous permettre de déterminer l'utilisation passée d'une clé AWS KMS donnée. Si vous souhaitez analyser des événements survenus il y a plus de 90 jours, vous pouvez créer un historique CloudTrail.



# Tâche 6 : revue de la rotation des clés

Il se peut que vous deviez effectuer une rotation de vos clés KMS en raison de règles commerciales ou contractuelles ou de réglementations gouvernementales. AWS KMS prend en charge la rotation automatique des clés uniquement pour les clés KMS à chiffrement symétrique créées par AWS KMS. La rotation automatique est facultative pour les clés KMS gérées par le client.

Au cours de cette tâche, vous allez passer en revue la fonctionnalité permettant d'activer la rotation automatique de la clé que vous avez créée dans cet atelier. Cependant, vous ne pouvez pas voir la rotation en action, car elle ne sera effective qu'au bout d'un an une fois activée.


Dans la console de gestion AWS, choisissez Key Management Service pour ouvrir la console AWS KMS.
Dans le volet de navigation de gauche, choisissez Clés gérées par le client.
Dans la liste, choisissez MyKMSKey, la clé que vous avez créée pour cet atelier.
Choisissez Rotation des clés.
Dans le menu Rotation des clés, sélectionnez Automatically rotate this KMS key every year (Effectuer une rotation automatique de cette clé KMs chaque année).
Sélectionnez Enregistrer.
Le message Rotation des clés mise à jour s'affiche en haut.

<-------------------------->


<img width="796" height="301" alt="image" src="https://github.com/user-attachments/assets/acff4d39-7897-4fb7-9572-003cc5e11742" />


<-------------------------->



<img width="813" height="358" alt="image" src="https://github.com/user-attachments/assets/20b4bf5b-2aa8-4af5-9b41-94ecc603802f" />



# Conclusion

Félicitations ! Vous avez réussi à effectuer les tâches suivantes :
passer en revue le chiffrement par défaut fourni par Amazon S3 ;
accéder à l'objet Amazon S3 chiffré ;
créer une clé AWS KMS gérée par le client pour chiffrer et déchiffrer des données au repos ;
créer et attacher un volume de données chiffré à une instance EC2 existante ;
désactiver et réactiver une clé AWS KMS, et observer les effets sur l'accès aux données ;
surveiller l'utilisation de la clé de chiffrement à l'aide de l'historique des événements CloudTrail ;
passer en revue la rotation des clés.
















