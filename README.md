

# 🌐 **Synthèse comparative — VMware ESXi, Proxmox VE et Incus**
*Comparaison technique des trois solutions étudiées dans le cadre du projet.*

L’infrastructure initialement envisagée reposait sur **VMware ESXi**, hyperviseur réputé pour sa **stabilité** et son **adoption dans le monde professionnel**.  
Au cours du projet, deux solutions alternatives ont été étudiées et déployées : **Proxmox VE**, environnement d’hypervision libre intégrant **virtualisation complète**, **conteneurs** et **capacités de clustering avancées**, et **Incus**, plateforme moderne de **conteneurisation système** dérivée de LXD.

Cette synthèse analyse les **fonctionnements**, **forces**, **limites** et **domaines d’usage** de ces trois technologies, afin de comprendre ce que chacune peut apporter — ou non — à une architecture virtuelle.

---

## **1. VMware ESXi — Un hyperviseur professionnel robuste et éprouvé**

VMware ESXi repose sur une architecture **bare‑metal**, où l’hyperviseur est installé directement sur le serveur **sans système d’exploitation intermédiaire**.  
Son fonctionnement repose exclusivement sur la **virtualisation complète** via le moteur **VMkernel**, reconnu pour sa **stabilité** et son **efficacité**.

ESXi se distingue par une excellente **gestion des ressources**, une grande **fiabilité**, et une intégration serrée avec l’écosystème VMware (**vCenter**, **vMotion**, **VSAN**, **HA**…).  
Cependant, bon nombre de fonctionnalités avancées — telles que la **haute disponibilité**, le **clustering**, ou la **migration à chaud** — nécessitent **vCenter**, une solution **payante**.

La dépendance aux **licences** et l’écosystème **propriétaire** limitent la flexibilité et augmentent le coût d’adoption, en particulier pour des environnements éducatifs ou expérimentaux.

**ESXi offre donc une plateforme extrêmement solide, mais parfois rigide et coûteuse.**

---

## **2. Proxmox VE — Une alternative complète, ouverte et polyvalente**

Proxmox VE combine dans un même environnement :

- un hyperviseur **KVM** pour la virtualisation complète,  
- **LXC** pour la virtualisation légère,  
- la gestion native du stockage **ZFS**,  
- un **clustering intégré**,  
- l’utilisation directe de **Ceph** pour le stockage distribué.

Son fonctionnement repose entièrement sur des technologies **open‑source** éprouvées.  
Contrairement à ESXi, Proxmox ne nécessite **aucun composant propriétaire** pour activer le **clustering**, la **migration de VM** ou la **haute disponibilité**.

L'interface web est **moderne**, **centralisée** et **intuitive**, ce qui facilite la gestion de plusieurs nœuds.  
La flexibilité de Proxmox est un atout : stockage hybride, intégration réseau avancée, gestion fine des conteneurs comme des machines virtuelles.

Sa seule limite réelle réside dans une **consommation de ressources plus élevée** que celle d’un hyperviseur spécialisé, ainsi qu’une architecture parfois plus **complexe** à maîtriser lorsque l’on combine **cluster**, **Ceph** et **ZFS**.

---

## **3. Incus — Conteneurisation système performante et flexible**

Incus est un système de **system containers**, orienté vers la **virtualisation légère**.  
Contrairement à ESXi et Proxmox, Incus ne propose **pas de virtualisation complète** : les conteneurs partagent **le noyau de l’hôte**.

Cette approche permet une **efficacité exceptionnelle** :  
- consommation **minimale** de ressources,  
- démarrage **instantané**,  
- déploiement **rapide en série**,  
- snapshots **quasi immédiats**.

Incus propose également des fonctionnalités avancées comme la **gestion de profils**, la **configuration réseau intégrée**, les **images cloud‑init**, et le **clustering multi‑nœuds**.

En revanche, Incus ne peut pas exécuter des systèmes nécessitant **leur propre kernel** (Windows, appliances virtuelles, etc.).  
Il ne remplace donc pas un **hyperviseur classique**.

Son usage est idéal pour des environnements **DevOps**, des environnements de **test**, ou des déploiements massifs de conteneurs système, mais pas pour des workloads nécessitant une **isolation matérielle complète**.


## Comparatif général des fonctionnalités

| Critère                | **VMware ESXi**                                   | **Proxmox VE**                   | **Incus**                        |
| ---------------------- | ------------------------------------------------- | -------------------------------- | -------------------------------- |
| Type de virtualisation | **Virtualisation complète**                       | **KVM (VM) + LXC (conteneurs)**  | **Conteneurs système**           |
| Licence                | Propriétaire (payant pour les fonctions avancées) | **Open‑source**                  | **Open‑source**                  |
| Clustering             | ✔ Avec vCenter (payant)                           | **✔ Intégré nativement**         | ✔ Clustering conteneurs          |
| Migration à chaud      | ✔ vMotion (payant)                                | **✔ Native, gratuite**           | N/A (pas de VM)                  |
| Stockage distribué     | ✔ VSAN (licence)                                  | **✔ Ceph intégré**               | Support externe possible         |
| Haute disponibilité    | ✔ Avec vCenter                                    | **✔ Proxmox HA (Ceph)**          | Limité aux conteneurs            |
| Performances           | Excellentes                                       | Très bonnes                      | **Exceptionnelles (conteneurs)** |
| Flexibilité            | Moyenne (écosystème fermé)                        | **Très élevée**                  | Très élevée                      |
| Support OS invités     | Tous OS                                           | Tous OS                          | Linux uniquement (même kernel)   |
| Idéal pour             | Production pro propriétaire                       | **Infra pro/éducative complète** | DevOps, labs, conteneurs         |


---

















# Serveur Incus - Document Technique

Projet de virtualisation — SAE / Infrastructure réseau

# 1. Mise en service physique du serveur

Pour initialiser le serveur, j’ai procédé à une prise en main locale. La machine était équipée d’un module ILO (Integrated Lights-Out) permettant une administration à distance, mais les identifiants avaient été perdus.

J’ai donc connecté localement :

un écran,

un clavier,

un câble réseau sur le port ILO.

Cela m’a permis d’accéder à l’interface BIOS du serveur, réinitialiser les identifiants ILO, et retrouver un contrôle complet de la machine.

## Réinitialisation et accès BIOS

J’ai restauré le mot de passe et le nom d’utilisateur ILO directement depuis le BIOS, permettant la reprise complète de l’administration du serveur.

# 2. Intégration réseau ILO

Le serveur devait être intégré dans notre réseau pédagogique, défini sur la plage :

10.202.0.0/16


Le premier serveur Proxmox avait déjà pour adresse :

10.202.4.1/16


J’ai attribué au module ILO une adresse disponible :

10.202.4.2/16


Cette configuration permettait une administration distante via navigateur :

https://10.202.4.2


📷 Capture à insérer : Administration ILO affichant l’adresse 10.202.4.2

# 3. Nettoyage du stockage (RAID matériel)

Avant d’installer un système d’exploitation propre, le serveur contenait plusieurs volumes RAID matériels historiques.
Je les ai supprimés via l’outil RAID du BIOS afin de repartir sur un stockage vierge.

## Objectifs

éviter les conflits lors de l’installation,

éliminer d’anciennes signatures de partitions,

garantir un système stable et propre.

📷 Capture à insérer : RAID avant / après suppression

# 4. Installation de Ubuntu Server

J’ai téléchargé l’image ISO Ubuntu Server sur mon poste, puis je l’ai montée virtuellement via l’interface ILO.
Le serveur a été configuré pour booter sur l’ISO et lancer l’installation.

Le système Ubuntu a été installé sur un seul disque, avec la structure recommandée :

Partition EFI
Partition /


📷 Capture à insérer : ILO affichant ISO monté

# 5. Configuration réseau du système Ubuntu

Une fois l’installation terminée, j’ai configuré la carte réseau principale en adressage statique.

Adresse IP   : 10.202.4.69/16
Passerelle   : 10.202.4.1
DNS          : 1.1.1.1 ou 8.8.8.8


Cette adresse est distincte de l’adresse ILO, car un contrôle physique (ILO) ne doit jamais partager la même identité réseau que l’OS principal.

Une fois la configuration appliquée, on pouvait accéder au serveur en SSH :

ssh titouan@10.202.4.69


📷 Capture à insérer : ip addr ou Netplan confirmant l’IP 10.202.4.69

6. Installation du démon Incus

Incus est un démon de virtualisation basé sur la technologie LXC/LXD. Il permet d’héberger :

des conteneurs Linux,

des machines virtuelles légères,

des environnements de test,

des snapshots et migrations.

L’installation demande la configuration du dépôt Zabbly, car Incus n’est pas présent dans les dépôts Ubuntu officiels.

## Procédure technique simplifiée
sudo apt update
sudo apt install incus
sudo incus admin init


📷 Capture à insérer : Installation Incus réussie dans le terminal

## Initialisation d’Incus

La commande suivante configure automatiquement :

le stockage,

le réseau virtuel,

l’accès distant,

et l’activation du démon.

sudo incus admin init


Incus devient ensuite pleinement opérationnel.

7. Déploiement de l’interface Incus UI (HTTPS)

Même si Incus est administrable entièrement en ligne de commande, une interface web permet de :

visualiser les conteneurs,

gérer les ressources,

lancer des commandes,

créer des snapshots,

surveiller l’activité réseau,

déployer des VM plus rapidement.

J’ai installé Incus UI, qui expose une interface HTTPS sur le port :

8443

## Sécurisation HTTPS par certificats

L’accès web nécessite la mise en place de certificats SSL.

### Sur le serveur (Incus)

génération d’un certificat .crt

ajout du certificat à Incus

activation de l’accès distant

### Sur le navigateur (Chrome)

importation d’un certificat .pfx

validation de la confiance locale

## Pare-feu Ubuntu (UFW)

Ouverture du port sécurisé :

sudo ufw allow 8443


📷 Capture à insérer : ufw status montrant le port 8443 ouvert

## Accès à Incus UI

Une fois les certificats installés :

https://10.202.4.69:8443/ui


📷 Capture à insérer : Incus UI fonctionnel dans le navigateur

8. Création d’instances (conteneurs et VM)

Depuis Incus UI, j’ai pu déployer :

2 conteneurs AlmaLinux

1 conteneur Devuan

1 machine virtuelle Linux Mint

Ces instances ont été créées et administrées directement depuis le navigateur, avec :

configuration réseau,

monitoring,

consoles interactives,

snapshots,

gestion des ressources.

📷 Capture à insérer : liste des instances dans Incus UI

9. Schéma d’architecture globale

Voici une représentation simplifiée de l’infrastructure :

                       ┌─────────────────────────────┐
                       │           Internet           │
                       └───────────────▲──────────────┘
                                       │
                         LAN 10.202.4.0/16
                                       │
              ┌─────────────────────────────────────┐
              │             SWITCH Réseau           │
              └───────▲────────────────────────────┘
                      │
          ┌───────────────────────────┐
          │       Serveur #1          │
          │         Proxmox           │
          │       10.202.4.1          │
          └───────────────────────────┘
                      │
          ┌───────────────────────────┐
          │       Serveur #2          │
          │  Ubuntu + Incus + UI      │
          │       10.202.4.69         │
          └───────────────────────────┘
                      │
    ┌──────────────────────────────────────────────┐
    │             Instances Incus                  │
    │  ┌────────────┬─────────────┬──────────────┐ │
    │  │ AlmaLinux   │ Devuan      │ Linux Mint   │ │
    │  └────────────┴─────────────┴──────────────┘ │
    └──────────────────────────────────────────────┘

10. Conclusion professionnelle

Ce projet m’a permis de déployer une plateforme complète de virtualisation basée sur Incus, depuis un serveur physique vierge, jusqu’à une administration graphique sécurisée en HTTPS.

J’ai manipulé :

administration matérielle via ILO,

configuration réseau statique,

suppression de RAID,

installation système Ubuntu,

installation du démon Incus,

configuration de certificats SSL,

mise en place du pare-feu,

déploiement d’instances conteneurs et VM depuis une interface web.

Cette infrastructure est aujourd’hui stable, moderne, automatisable, et adaptée à des usages pédagogiques, exploratoires ou applicatifs.
