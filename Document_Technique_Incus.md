---
title: Untitled

---

# Document Technique - Incus Titouan Mouysset 
# 1. Prise en main physique

La mise en service du serveur a commencé par une intervention physique indispensable. Le module ILO, normalement utilisé pour administrer la machine à distance, était verrouillé car les identifiants d’accès avaient été perdus. J’ai donc connecté un clavier, un écran et un câble réseau directement sur la machine afin d’accéder au BIOS.

Depuis l’interface de gestion, j’ai procédé à la réinitialisation complète des identifiants ILO, rendant à nouveau possible l’administration distante sécurisée. Cette étape a permis de reprendre le contrôle matériel de l’équipement avant toute installation logicielle. En modifiant directement les identifiants dans le bios.



# 2. Intégration d’ILO dans l’infrastructure réseau

L’objectif suivant consistait à intégrer le serveur dans le réseau pédagogique, construit sur le plan d’adressage 10.202.0.0/16.
Le premier serveur Proxmox, déjà opérationnel, utilisait l’adresse 10.202.4.1. Afin de suivre une logique cohérente, j’ai configuré ILO avec l’adresse 10.202.4.2/16.

Une fois l’interface réseau ILO paramétrée, l’administration à distance via un navigateur est devenue possible à l’adresse suivante :
https://10.202.4.2



# 3. Réinitialisation complète des volumes RAID

Avant d’installer un nouveau système, il était impératif de nettoyer l’infrastructure de stockage. Le serveur contenait plusieurs volumes RAID matériels, susceptibles d’interférer avec l’installation ou de générer des conflits de partitions.

J’ai supprimé successivement tous les volumes RAID depuis l’outil de gestion dédié dans le bios, et réinitialisé chaque disque. Cette opération garantit un environnement de stockage sain et parfaitement maîtrisé pour accueillir Ubuntu Server avec Incus.



# 4. Installation d’Ubuntu Server via l’interface ILO

L’installation d’Ubuntu Server a été réalisée entièrement à distance grâce à la fonctionnalité Virtual Media d’ILO.
J’ai téléchargé l’image ISO depuis mon poste, l’ai montée virtuellement dans le serveur, puis forcé le boot sur ce support.

L’assistant d’installation d’Ubuntu a été configuré pour utiliser un seul disque avec une structure moderne composée d’une partition EFI et d’un système racine /. Ce schéma assure une compatibilité optimale avec UEFI et permet une maintenance simplifiée.


# 5. Configuration réseau du système Ubuntu

Une fois Ubuntu installé, j’ai configuré son interface réseau en adressage statique afin de l’intégrer pleinement à l’infrastructure existante.
Le serveur Ubuntu utilise désormais l’adresse 10.202.4.69/16, distincte de l’adresse ILO pour garantir un cloisonnement logique entre :

la gestion matérielle (ILO),

et l’administration du système d’exploitation.

La passerelle a été fixée sur 10.202.4.1, et les serveurs DNS configurés pour assurer une résolution fiable.
Cette configuration m’a permis d’accéder au serveur via SSH, ce qui a marqué la transition vers l’administration entièrement distante.



# 6. Installation et initialisation du démon Incus

Avec un serveur fonctionnel, j’ai installer Incus, un moteur de virtualisation moderne basé sur LXC/LXD.
Incus n’étant pas disponible dans les dépôts Ubuntu par défaut, j’ai ajouté le dépôt officiel Zabbly, permettant de récupérer une version maintenue et stable du démon.

Après installation, la commande d’initialisation a automatisé la création du stockage, du pont réseau virtuel et des paramètres de sécurité. À l’issue de cette phase, Incus était opérationnel et prêt à héberger des conteneurs ou des machines virtuelles.


# 7. Mise en place de l’interface graphique sécurisée Incus UI

Bien qu’Incus puisse être entièrement piloté via la ligne de commande, j’ai choisi d’installer Incus UI, une interface web qui améliore considérablement la gestion des ressources virtuelles.

Incus UI fonctionne exclusivement en HTTPS, ce qui a nécessité la génération d’un certificat serveur (.crt) et son ajout au trust store d’Incus. Une seconde étape a consisté à importer un certificat utilisateur (.pfx) dans mon navigateur afin d’établir une connexion sécurisée et authentifiée.

Cette mécanique de certificats permet d’assurer un haut niveau de sécurité, aligné avec les pratiques des environnements professionnels.



# 8. Accès sécurisé à l’interface Incus UI

Une fois les certificats installés et le pare-feu reconfiguré pour autoriser les communications sur le port 8443, l’interface graphique est devenue accessible via :
https://10.202.4.69:8443/ui

Cela a marqué la transition vers une gestion visuelle complète, facilitant la création, le suivi et la maintenance des instances.



# 9. Création des premières instances virtuelles

Depuis Incus UI, j’ai créé mes premières instances afin de valider le fonctionnement de la plateforme :

deux conteneurs AlmaLinux, adaptés aux environnements de type entreprise,

un conteneur Devuan, reconnu pour sa légèreté et l’absence de systemd,

une machine virtuelle Linux Mint, permettant de tester une interface graphique complète.

Chaque instance a été configurée avec un stockage dédié, un réseau virtuel stable et des ressources adaptées. La mise en œuvre s’est révélée fluide grâce à la structure modulaire d’Incus.


# 10. Architecture finale du système

L’architecture globale déployée reflète une infrastructure moderne, sécurisée et segmentée, où chaque niveau — matériel, OS, démon de virtualisation, interface graphique et instances — coopère de manière cohérente.


# Conclusion professionnelle

Ce projet m’a permis de déployer une solution de virtualisation complète, depuis la remise en service matérielle du serveur jusqu’à la gestion avancée via une interface web sécurisée.
J’ai configuré l’ensemble de l’infrastructure réseau, réinitialisé les systèmes RAID, installé Ubuntu Server, mis en place Incus et son interface graphique, puis créé mes premières instances en conteneurs et en VM.

Ce travail m’a confronté à des problématiques réelles d’administration système : gestion du matériel, configuration réseau, sécurité HTTPS, orchestration virtuelle.
Le résultat est une plateforme propre, performante, professionnelle et totalement exploitable pour un usage pédagogique ou applicatif.