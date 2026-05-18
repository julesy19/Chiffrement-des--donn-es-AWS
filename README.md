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



