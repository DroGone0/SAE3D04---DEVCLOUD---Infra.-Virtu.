---
title: Document Technique - Déploiement Incus
author: Titouan Mouysset
---

#  Document Technique - Incus
## Titouan Mouysset

> **Objectif** : Déploiement d'une infrastructure de virtualisation complète avec Incus sur serveur physique

---

##  1. Prise en main physique

La mise en service du serveur a commencé par une intervention physique **indispensable**. Le module **ILO** (Integrated Lights-Out), normalement utilisé pour administrer la machine à distance, était verrouillé car les identifiants d'accès avaient été perdus. 

> **Problématique** : Accès ILO verrouillé - nécessité d'intervention physique

J'ai donc connecté un clavier, un écran et un câble réseau directement sur la machine afin d'accéder au **BIOS**.

<div align="center">
<img src="Image_SAED04_Incus/imageBiosResetMDP.jpg" alt="Écran de démarrage BIOS - Réinitialisation mot de passe" width="600" style="border-radius: 5px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); margin: 10px 0;">
</div>

Depuis l'interface de gestion, j'ai procédé à la **réinitialisation complète des identifiants ILO**, rendant à nouveau possible l'administration distante sécurisée. Cette étape a permis de reprendre le contrôle matériel de l'équipement avant toute installation logicielle.

<div align="center">
<img src="Image_SAED04_Incus/image_EcranDemarrageBiosIP.jpg" alt="Écran de démarrage BIOS - Configuration IP" width="600" style="border-radius: 5px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); margin: 10px 0;">
</div>

>  **Résultat** : Contrôle matériel restauré - Administration ILO opérationnelle

---

## 2. Intégration d'ILO dans l'infrastructure réseau

L'objectif suivant consistait à intégrer le serveur dans le réseau pédagogique, construit sur le plan d'adressage **10.202.0.0/16**.

Le premier serveur Proxmox, déjà opérationnel, utilisait l'adresse **10.202.4.1**. Afin de suivre une logique cohérente, j'ai configuré ILO avec l'adresse **10.202.4.2/16**.

### Configuration réseau ILO

<div align="center">
<img src="Image_SAED04_Incus/Image_IP_ManuelleBrancher.png" alt="Configuration IP manuelle - Câble branché" width="500" style="border-radius: 5px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); margin: 10px 0;">
<img src="Image_SAED04_Incus/Image_Ip_ManuelleNonbrancher.png" alt="Configuration IP manuelle - Câble non branché" width="500" style="border-radius: 5px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); margin: 10px 0;">
</div>

Une fois l'interface réseau ILO paramétrée, l'administration à distance via un navigateur est devenue possible à l'adresse suivante :

```bash
https://10.202.4.2
```


---

##  3. Réinitialisation complète des volumes RAID

Avant d'installer un nouveau système, il était **impératif** de nettoyer l'infrastructure de stockage. Le serveur contenait plusieurs volumes RAID matériels, susceptibles d'interférer avec l'installation ou de générer des conflits de partitions.

### Processus de réinitialisation RAID

J'ai supprimé successivement tous les volumes RAID depuis l'outil de gestion dédié dans le BIOS, et réinitialisé chaque disque. Cette opération garantit un environnement de stockage sain et parfaitement maîtrisé pour accueillir Ubuntu Server avec Incus.

#### Étape 1 : Réinitialisation du RAID matériel
<div align="center">
<img src="Image_SAED04_Incus/Image_Etape1_resetRaidmateriel.png" alt="Étape 1 - Réinitialisation RAID matériel" width="550" style="border-radius: 5px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); margin: 10px 0;">
</div>

#### Étape 2 : Sélection des SSD à réinitialiser
<div align="center">
<img src="Image_SAED04_Incus/Image_Etape2_MesSSd%20a%20reset.png" alt="Étape 2 - SSD à réinitialiser" width="550" style="border-radius: 5px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); margin: 10px 0;">
</div>

#### Étape 3 : Confirmation de réinitialisation RAID
<div align="center">
<img src="Image_SAED04_Incus/Image_Etape3_RaidMaterielareset.png" alt="Étape 3 - RAID matériel à réinitialiser" width="550" style="border-radius: 5px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); margin: 10px 0;">
</div>

#### Étape 4 : Suppression du RAID matériel
<div align="center">
<img src="Image_SAED04_Incus/Image_Etape4_DeleteAidMateriel.png" alt="Étape 4 - Suppression RAID matériel" width="550" style="border-radius: 5px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); margin: 10px 0;">
</div>

#### Étape 5 : Finalisation de la réinitialisation
<div align="center">
<img src="Image_SAED04_Incus/Image_Etape5_ResetRaid2.png" alt="Étape 5 - Finalisation réinitialisation RAID" width="550" style="border-radius: 5px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); margin: 10px 0;">
</div>

#### Configuration finale : Mode JBOD
<div align="center">
<img src="Image_SAED04_Incus/Image_SSD_JBOD.png" alt="Configuration SSD en mode JBOD" width="550" style="border-radius: 5px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); margin: 10px 0;">
</div>

>  **Résultat** : Environnement de stockage propre et prêt pour l'installation

---

##  4. Installation d'Ubuntu Server via l'interface ILO

L'installation d'Ubuntu Server a été réalisée **entièrement à distance** grâce à la fonctionnalité **Virtual Media** d'ILO.

### Processus d'installation

#### Montage de l'ISO Ubuntu Server
J'ai téléchargé l'image ISO depuis mon poste, l'ai montée virtuellement dans le serveur, puis forcé le boot sur ce support.

<div align="center">
<img src="Image_SAED04_Incus/Image_Monter_Iso.png" alt="Montage de l'ISO Ubuntu Server" width="500" style="border-radius: 5px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); margin: 10px 0;">
</div>

#### Démarrage du serveur
<div align="center">
<img src="Image_SAED04_Incus/Image_Serveur_Lancement.png" alt="Lancement du serveur" width="500" style="border-radius: 5px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); margin: 10px 0;">
</div>

#### Boot sur le bon support
<div align="center">
<img src="Image_SAED04_Incus/Image_BooterSurlebon.png" alt="Boot sur le support virtuel" width="500" style="border-radius: 5px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); margin: 10px 0;">
</div>

#### Retrait du média virtuel
<div align="center">
<img src="Image_SAED04_Incus/Image_Moment_EnelverMedia_RedemarrageServeur.png" alt="Moment pour retirer le média virtuel" width="500" style="border-radius: 5px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); margin: 10px 0;">
</div>

### Configuration du disque

L'assistant d'installation d'Ubuntu a été configuré pour utiliser un seul disque avec une structure moderne composée d'une partition **EFI** et d'un système racine **/**.

<div align="center">
<img src="Image_SAED04_Incus/Image_ConfigSSD1_Ubuntuserveur.png" alt="Configuration du premier SSD Ubuntu Server" width="500" style="border-radius: 5px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); margin: 10px 0;">
<img src="Image_SAED04_Incus/Image_CustomStorageconfigubuntu.png" alt="Configuration personnalisée du stockage Ubuntu" width="500" style="border-radius: 5px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); margin: 10px 0;">
</div>

>  **Avantages** : Compatibilité UEFI optimale - Maintenance simplifiée

---

##  5. Configuration réseau du système Ubuntu

Une fois Ubuntu installé, j'ai configuré son interface réseau en **adressage statique** afin de l'intégrer pleinement à l'infrastructure existante.

### Paramètres réseau

Le serveur Ubuntu utilise désormais l'adresse **10.202.4.69/16**, distincte de l'adresse ILO pour garantir un cloisonnement logique entre :

-  **la gestion matérielle (ILO)** : `10.202.4.2`
-  **l'administration du système d'exploitation** : `10.202.4.69`

<div align="center">
<img src="Image_SAED04_Incus/Image_Configuration%20UbuntuServer.png" alt="Configuration Ubuntu Server" width="500" style="border-radius: 5px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); margin: 10px 0;">
<img src="Image_SAED04_Incus/Image_Ubuntuserveur_configurer.png" alt="Configuration avancée Ubuntu Server" width="500" style="border-radius: 5px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); margin: 10px 0;">
<img src="Image_SAED04_Incus/image_MDP_User_Configuration_Serveur.png" alt="Configuration utilisateur et mot de passe" width="500" style="border-radius: 5px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); margin: 10px 0;">
</div>

### Configuration réseau finale

```yaml
Adresse IP:     10.202.4.69/16
Passerelle:     10.202.4.1
DNS:            Configurés pour résolution fiable
```

Cette configuration m'a permis d'accéder au serveur via **SSH**, ce qui a marqué la transition vers l'administration entièrement distante.

>  **Résultat** : Administration distante opérationnelle via SSH

---

##  6. Installation et initialisation du démon Incus

Avec un serveur fonctionnel, j'ai installé **Incus**, un moteur de virtualisation moderne basé sur **LXC/LXD**.

### Installation depuis le dépôt Zabbly

Incus n'étant pas disponible dans les dépôts Ubuntu par défaut, j'ai ajouté le dépôt officiel **Zabbly**, permettant de récupérer une version maintenue et stable du démon.

```bash
# Ajout du dépôt Zabbly
sudo add-apt-repository ppa:zabbly/incus-stable

# Installation d'Incus
sudo apt update
sudo apt install incus
```

### Initialisation d'Incus

Après installation, la commande d'initialisation a automatisé la création du stockage, du pont réseau virtuel et des paramètres de sécurité.

<div align="center">
<img src="Image_SAED04_Incus/Image_Lancement_Incus_Config.png" alt="Lancement de la configuration Incus" width="600" style="border-radius: 5px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); margin: 10px 0;">
</div>

À l'issue de cette phase, Incus était opérationnel et prêt à héberger des conteneurs ou des machines virtuelles.

>  **Résultat** : Démon Incus opérationnel - Prêt pour la virtualisation

---

##  7. Mise en place de l'interface graphique sécurisée Incus UI

Bien qu'Incus puisse être entièrement piloté via la ligne de commande, j'ai choisi d'installer **Incus UI**, une interface web qui améliore considérablement la gestion des ressources virtuelles.

### Configuration HTTPS

Incus UI fonctionne exclusivement en **HTTPS**, ce qui a nécessité :

1. **Génération d'un certificat serveur** (`.crt`) et son ajout au trust store d'Incus
2. **Import d'un certificat utilisateur** (`.pfx`) dans mon navigateur afin d'établir une connexion sécurisée et authentifiée

Cette mécanique de certificats permet d'assurer un **haut niveau de sécurité**, aligné avec les pratiques des environnements professionnels.

>  **Sécurité** : Authentification par certificats - Communication HTTPS chiffrée

---

##  8. Accès sécurisé à l'interface Incus UI

Une fois les certificats installés et le pare-feu reconfiguré pour autoriser les communications sur le port **8443**, l'interface graphique est devenue accessible via :

```bash
https://10.202.4.69:8443/ui
```

> **Accès** : Interface graphique opérationnelle - Gestion visuelle complète

Cela a marqué la transition vers une gestion visuelle complète, facilitant la création, le suivi et la maintenance des instances.

---

##  9. Création des premières instances virtuelles

Depuis Incus UI, j'ai créé mes premières instances afin de valider le fonctionnement de la plateforme :

### Instances déployées

| Instance | Type | Description |
|----------|------|-------------|
| **AlmaLinux 1** | Conteneur | Environnement de type entreprise |
| **AlmaLinux 2** | Conteneur | Environnement de type entreprise |
| **Devuan** | Conteneur | Distribution légère sans systemd |
| **Linux Mint** | Machine virtuelle | Interface graphique complète |

### Configuration des instances

Chaque instance a été configurée avec :
-  **Stockage dédié**
-  **Réseau virtuel stable**
-  **Ressources adaptées** (CPU, RAM)

La mise en œuvre s'est révélée fluide grâce à la structure modulaire d'Incus.



## 🏗️ 10. Architecture finale du système

L'architecture globale déployée reflète une infrastructure **moderne**, **sécurisée** et **segmentée**, où chaque niveau coopère de manière cohérente :

```
┌─────────────────────────────────────────────────────────┐
│                    Infrastructure Réseau                 │
│                     10.202.0.0/16                        │
└─────────────────────────────────────────────────────────┘
                            │
        ┌───────────────────┴───────────────────┐
        │                                       │
┌───────▼────────┐                    ┌────────▼────────┐
│   Serveur      │                    │   Serveur       │
│   Proxmox      │                    │   Ubuntu        │
│  10.202.4.1    │                    │  10.202.4.69    │
└────────────────┘                    └─────────────────┘
                                              │
                                    ┌─────────▼─────────┐
                                    │   ILO Management  │
                                    │   10.202.4.2      │
                                    └───────────────────┘
                                              │
                                    ┌─────────▼─────────┐
                                    │   Incus Daemon    │
                                    │   Port: 8443      │
                                    └───────────────────┘
                                              │
                    ┌─────────────────────────┼─────────────────────────┐
                    │                         │                         │
            ┌───────▼────────┐        ┌───────▼────────┐        ┌───────▼────────┐
            │  AlmaLinux 1   │        │   Devuan       │        │  Linux Mint    │
            │  (Conteneur)   │        │  (Conteneur)   │        │     (VM)       │
            └────────────────┘        └────────────────┘        └────────────────┘
```


## Conclusion professionnelle

Ce projet m'a permis de déployer une solution de virtualisation **complète**, depuis la remise en service matérielle du serveur jusqu'à la gestion avancée via une interface web sécurisée.


### Résultats obtenus

J'ai configuré l'ensemble de l'infrastructure réseau, réinitialisé les systèmes RAID, installé Ubuntu Server, mis en place Incus et son interface graphique, puis créé mes premières instances en conteneurs et en VM.

Ce travail m'a confronté à des problématiques réelles d'administration système : gestion du matériel, configuration réseau, sécurité HTTPS, orchestration virtuelle.

---

**Document rédigé par** : Titouan Mouysset  
**Date** : 2025
**Technologies** : Ubuntu Server, Incus, ILO, LXC/LXD
