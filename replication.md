# Redondance Active Directory : réplication et tolérance de panne
## Contexte

Ce projet prolonge le labo Active Directory précédent (entreprise-active-directory-lab) en y ajoutant un deuxième contrôleur de domaine. L'objectif était de reproduire une situation courante en entreprise : garantir que l'authentification des utilisateurs continue de fonctionner même si l'un des serveurs tombe en panne.

Environnement technique
Contrôleur de domaine 1 : Windows Server 2022, IP 192.168.1.10
Contrôleur de domaine 2 : Windows Server 2022, IP 192.168.1.11
Poste client : Windows 10 x64, joint au domaine entreprise.local
VMware Workstation
Étapes réalisées
### 1. Ajout du deuxième contrôleur de domaine

Une deuxième machine virtuelle Windows Server 2022 a été installée, avec une adresse IP fixe (192.168.1.11) et son DNS pointé vers le premier serveur (192.168.1.10) pour pouvoir localiser et rejoindre le domaine existant. Le serveur a ensuite été joint au domaine entreprise.local, puis promu en contrôleur de domaine additionnel via le rôle Active Directory Domain Services, en choisissant l'option "Ajouter un contrôleur de domaine à un domaine existant" plutôt que la création d'une nouvelle forêt.

### 2. Vérification de la réplication

Une fois la promotion terminée, la console Utilisateurs et ordinateurs Active Directory a été ouverte sur ce deuxième serveur. Les unités d'organisation Comptabilité et Direction, ainsi que les utilisateurs Marc Henry et Jean Brow, y apparaissaient automatiquement, sans aucune recréation manuelle. Cette synchronisation confirme que la réplication Active Directory entre les deux contrôleurs de domaine fonctionne correctement.

### 3. Test de tolérance de panne

Le premier contrôleur de domaine (192.168.1.10) a été volontairement éteint pour simuler une panne. Une première tentative de connexion depuis le poste client a échoué : la commande gpupdate /force a renvoyé une erreur d'absence de connectivité vers un contrôleur de domaine.

L'analyse a montré que le poste client n'avait comme DNS que l'adresse du premier serveur, désormais injoignable. Le DNS secondaire (192.168.1.11) a été ajouté dans la configuration réseau du poste, ce qui a résolu le problème.

Un nouveau test de connexion, cette fois avec le compte de Jean Brow, a confirmé le bon fonctionnement de la bascule : le gpresult /r a affiché "Stratégie de groupe appliquée depuis : WIN-BN5K50VTBT7.entreprise.local", soit le nom du deuxième serveur, alors que le premier restait éteint.

### Difficulté rencontrée

La réplication entre les deux contrôleurs de domaine fonctionnait dès le départ, mais le poste client restait dépendant d'un seul serveur DNS. Ce point a permis de bien comprendre une limite pratique de la redondance Active Directory : la présence d'un deuxième contrôleur de domaine ne suffit pas à elle seule, il faut aussi que chaque poste client soit configuré avec un DNS secondaire pour pouvoir réellement basculer dessus en cas de panne du premier.

### Résultat

Une infrastructure Active Directory redondante, avec deux contrôleurs de domaine synchronisés et un test de bascule réussi : un poste client reste capable de s'authentifier et de recevoir ses stratégies de groupe même lorsque le premier serveur est indisponible, à condition d'avoir configuré un DNS secondaire côté client.

## Prochaine étape

Documenter également la remise en service du premier serveur et vérifier que la réplication se remet automatiquement à jour dans les deux sens, ou introduire une première brique cloud pour amorcer un scénario d'infrastructure hybride.
