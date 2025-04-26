# FT (Fault Tolerance)

## F.T. 🧠 Définition

La fonctionnalité **vSphere FT** (Fault Tolerance) offre une protection de continuité d’activité que n’assure pas vSphere HA.  
FT fonctionne en créant **une copie dupliquée** (secondaire) d'une machine virtuelle sur un hôte différent et en **synchronisant** en permanence les deux machines virtuelles.

👉 En cas de défaillance d’un hôte ESXi, **la machine virtuelle secondaire prend immédiatement le relais** sans perte de données ni interruption.

**Utilisations typiques** :
- Applications stratégiques devant être disponibles à tout moment.
- Applications qui ne peuvent pas être mises en cluster nativement (système ou applicatif).

**Limitations principales** :
- **Augmentation de l’utilisation des ressources** : la machine secondaire consomme autant de ressources que la machine primaire (ressources ×2).
- **Restrictions sur les VMs protégées** :
  - Nombre limité de vCPU et de mémoire.
  - Pas de snapshots autorisés.
- **Protection au niveau VM uniquement** : si le système d'exploitation ou l'application crashent sur la machine primaire, la machine secondaire sera également impactée.


## F.T. 🛠️ Prérequis

La mise en place de la fonctionnalité **Fault Tolerance** nécessite :

- ✅ 2 hôtes **ESXi** minimum
- ✅ 1 instance **vCenter** avec un cluster configuré
- ✅ Licences **éligibles à FT** sur vCenter et les hôtes
- ✅ Adressage IP **statique** pour tous les hôtes
- ✅ Configuration réseau identique, avec :
  - Un port **VMkernel** pour le **Fault Tolerance logging**
  - Un port **VMkernel** pour **vMotion**
- ✅ **Stockage partagé** (accessible par tous les hyperviseurs)
- ✅ **Processeurs récents** supportant la virtualisation :
  - Intel : Penryn, Nehalem, Westmere, Sandy Bridge
  - AMD : Barcelona
- ✅ Activation de la fonctionnalité **HA**
- ✅ Disques virtuels des VMs au format **Thick**
- 🚫 Pas de **snapshots** sur les machines virtuelles protégées
<br>
<br>

# Configuration de la fonctionnalité FT (Fault Tolerance)

## 1. Activation du FT sur les hôtes ESXi

Tout d’abord, il faut activer la fonctionnalité **Fault Tolerance** sur l’adaptateur **VMKernel** :

- Aller sur :  
  `ESXi_1 > Monitor > Adaptateur VMkernel > vmk0 > Fault Tolerance`
- 🔁 Répéter **la même manipulation sur les deux hôtes ESXi** afin d’assurer une bonne communication et un basculement optimal.


## 2. Activation du FT sur une machine virtuelle

Sur la VM concernée :

- Clic droit sur la VM > **Fault Tolerance > Activer**
- Sélectionner la **banque de données** et **choisir l’ESXi secondaire**.

On peut ensuite observer que la fonction **Fault Tolerance** est bien activée sur la VM.

> ⚠️ **Important** : il est nécessaire d’installer **VMware Tools** sur la VM pour que vCenter puisse surveiller correctement son état.


<br>
<br>

![activation_fr](../images/ft_screen/choix_espace_stockage.png)
![activation_fr](../images/ft_screen/choix_host_vm_secondary.png)
![activation_fr](../images/ft_screen/installation_secondary_vm_ok.png)


## 3. Partie Test

Pour tester le bon fonctionnement de FT :

- Mettre en route la VM protégée par Fault Tolerance.
- **Éteindre** (`shut down`) l’ESXi où se trouve la machine virtuelle primaire.

Résultat :
- La VM secondaire passe automatiquement en **primaire** sur le second ESXi, **sans aucune coupure** pour l'utilisateur.
<br>


![test_ft](../images/ft_screen/after_ft_ESXI1_vm_secondary.png)
![test_ft](../images/ft_screen/before_ft_ESXI1_vm_primary.png)

## 4. Avantages et inconvénients de FT

### ✅ Avantages

- Continuité d'activité immédiate, sans interruption.
- Idéal pour les applications critiques.

### ⚠️ Inconvénients

- Pas de possibilité de **snapshots** (car les sauvegardes utilisent des snapshots).
- **Consommation doublée** des ressources (la VM secondaire est toujours prête à prendre le relais).



## 5. Cas d’utilisation typiques

- **Base de données SQL Server** utilisée par une application de commerce électronique.
- **Serveur web** hébergeant un site de vente pendant des pics de trafic (soldes, fêtes de fin d'année).
- **Système de gestion des dossiers médicaux** dans un hôpital, où l’accès instantané aux données est vital.


