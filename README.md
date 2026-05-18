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







