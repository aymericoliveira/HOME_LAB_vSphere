# 📦 Projet : Sauvegarde et Restauration de VM avec Veeam Backup & Replication

## 🧠 Objectif

Mettre en place une solution de sauvegarde et de restauration complète d'une machine virtuelle dans un environnement de lab local avec :
- Création d’un espace de stockage iSCSI sur FreeNAS
- Intégration dans Veeam Backup & Replication
- Sauvegarde complète d’une VM
- Restauration de la VM

Ce projet n’est **pas un tutoriel**, mais un **retour d’expérience** avec captures d’écran et notes personnelles.



## 🖥️ Infrastructure du Lab Veeam Backup & Replication

### Répartition des VMs

- **FreeNAS** : serveur de stockage iSCSI  
- **VM vCenter** : gestion de l’hyperviseur ESXi  
- **VM Windows Server (avec Veeam B&R)** : serveur de sauvegarde  
- **VM Linux** : VM à sauvegarder/restaurer  



## 🧱 Étape 1 - Création du stockage LUN iSCSI sur FreeNAS

La création du LUN iSCSI a été documentée dans un autre projet. Pour plus de détails, consultez [cette section](./Installation.md#section-cible-iscsi).

Une fois la configuration terminée sur FreeNAS, il suffit de monter l'espace disque sur l'hôte qui héberge Veeam.

![Montage du disque](./images/veeam_screen/montage_disque_v.png)  
![Partitionnement du disque](./images/veeam_screen/partition_disk_iscsi_san.png)



## 🖧 Étape 2 - Connexion de l’iSCSI à la VM Veeam

### Connexion depuis Windows Server (Veeam)
1. Ouvrir "Initiateur iSCSI"
2. Ajouter l’IP du target FreeNAS
3. Détection automatique du disque
4. Mise en ligne du disque
5. Initialisation + formatage du disque (NTFS)

![Connexion iSCSI](./images/veeam_screen/connexion_reseau_veeam_freenass_1.Png)



## ⚙️ Étape 3 - Installation de Veeam Backup & Replication

Installation de Veeam Backup & Replication sur le serveur Windows Server.

![Installation Veeam](./images/veeam_screen/installation_veeam_1.png)

Connexion à la console via un compte administrateur disposant des droits nécessaires.

![Accès à la console](./images/veeam_screen/installation_veeam_2.png)



## 📂 Étape 4 - Ajout du Backup Repository

Un **backup repository** est un espace de stockage utilisé pour enregistrer les sauvegardes.  
J’ai choisi un LUN iSCSI pour simuler une baie SAN professionnelle.

1. Aller dans "Backup Infrastructure"
2. Clic droit > *Add Backup Repository*

![Création du repository](./images/veeam_screen/add_backup_repository_1.png)

Nom et description :

![Nom du repository](./images/veeam_screen/add_backup_repository_2.png)

Scan des disques, puis sélection du LUN iSCSI :

![Sélection iSCSI](./images/veeam_screen/selection_repo.png)

Résumé de la configuration :

![Résumé](./images/veeam_screen/repo_ok.png)



### 🔧 Création du Proxy Veeam

**Backup Proxy** : il agit comme un moteur de transfert, gère le trafic de données entre source et cible, et améliore les performances.

![Backup Proxy créé](./images/veeam_screen/backup_proxy_done.png)

> ⚠️ Sans backup proxy, aucune sauvegarde ne peut être effectuée dans Veeam Backup & Replication.



## 🔗 Étape 5 - Connexion au vCenter dans Veeam

### Ajout du serveur vCenter

1. "Virtual Infrastructure" > *Add Server* > *VMware vSphere*  
2. Entrer l’adresse IP ou le FQDN du vCenter + identifiants

![Connexion au vCenter](./images/veeam_screen/connexion_vcenter_2.png)

### Ajout d’un compte administrateur

Ajout d’un utilisateur ayant les privilèges administrateurs :

![Ajout utilisateur](./images/veeam_screen/id_user_.png)

### Validation du certificat SSL

Validation du certificat du vCenter :

![Validation SSL](./images/veeam_screen/certificat_ssl.png)



## 💾 Étape 6 - Création d’un Job de Sauvegarde

Je crée un job de sauvegarde de la VM **Linux_Veeam** sur l’hôte **ESXi_2**.

1. "Home" > *Backup Job* > *Virtual Machine*
2. Nommer le job

![Nom du job](./images/veeam_screen/backup_job_1.png)

Sélection de la VM à sauvegarder :

![Choix de la VM](./images/veeam_screen/backup_job_2.png)

### 📌 Choix du mode de sauvegarde

J’ai opté pour une **sauvegarde incrémentale** après une sauvegarde complète.

> ✅ **Avantage** : rapide, moins de charge disque  
> ❌ **Inconvénient** : nécessite plusieurs points pour une restauration complète

![Sauvegarde incrémentale](./images/veeam_screen/backup_incremental.png)

### 🗃️ Configuration du proxy, repository et politique de rétention

- Proxy : auto
- Repository : iSCSI
- Rétention : **7 jours**

![Configuration du job](./images/veeam_screen/config_backup_vm.png)  
![Résumé du job](./images/veeam_screen/backup_job_3.png)

### ✅ Sauvegarde terminée avec succès

![Sauvegarde terminée](./images/veeam_screen/sauvegarde_full_vm_done.png)  
![Sauvegarde terminée (suite)](./images/veeam_screen/sauvegarde_full_vm_done_2.png)



## 🔁 Étape 7 - Restauration de la VM

Maintenant que la sauvegarde est faite, je procède à la **restauration complète** grâce à *Full VM Recovery*.

1. Clic droit sur la VM > *Restore entire VM*

![Lancement de la restauration](./images/veeam_screen/restauration_2.png)  
![Choix de la restauration](./images/veeam_screen/restore_vm_1.png)  
![Etapes de la restauration](./images/veeam_screen/restore_vm_2.png)

Je sélectionne la VM à restaurer :

![Sélection de la VM](./images/veeam_screen/select_vm_restore.png)

### 🟢 Résultat : succès

Restauration complète avec succès ✅

![Succès](./images/veeam_screen/restauration_successfully_1.png)

Dans l'historique, la VM apparaît bien restaurée :

![Historique](./images/veeam_screen/restauration_successfully_2.png)

*(Oui… il m’a fallu quelques essais 😅)*



## 📝 Conclusion

- ✅ Projet réalisé avec succès sur un lab local
- 💾 Sauvegarde incrémentale + rétention de 7 jours
- 🔁 Restauration complète fonctionnelle
- 🔒 Utilisation d’iSCSI pour simuler une baie SAN
- 💡 Expérience enrichissante dans la gestion de la sauvegarde VM
