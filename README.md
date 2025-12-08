# 🌐 SAE D04 — Étude comparative des solutions de virtualisation  
> [!NOTE]
> Projet réalisé dans le cadre de la SAE D04 : étude, installation, comparaison technique et financière de deux solutions de virtualisation parmi Proxmox, Hyper-V et VMware VSAN.

---

# 1. 🎯 Introduction

## 1.1 Contexte professionnel
Résumé synthétique de la situation de l’entreprise et des objectifs du chef d’équipe.

## 1.2 Objectifs de la SAE  
- Étudier deux solutions de virtualisation.  
- Installer et configurer ces solutions sur l’infrastructure DELL.  
- Tester et analyser le stockage distribué (CEPH, Corosync, VSAN).  
- Réaliser un comparatif technique et financier.  
- Produire un avis de synthèse pour un choix final.

## 1.3 Organisation du groupe  
- Liste des membres  
- Répartition générale des rôles  

---

# 2. 🗂️ Méthodologie & Outils de gestion de projet

> [!TIP]
> Cette section explique comment vous avez organisé le travail, réparti les tâches et documenté l’évolution du projet.

## 2.1 Méthodologie adoptée  
Exemple : Kanban, Scrum léger, cycles courts, milestones…

## 2.2 Utilisation de JIRA  
- Gestion des tickets  
- Tableau(x) Kanban  
- Workflow adopté  
- Captures d’écran des tableaux si nécessaire

## 2.3 Utilisation de GitHub  
- Dépôt du projet  
- Organisation des fichiers  
- Stratégie de commit / branches  
- Lien vers le repository :  
`https://github.com/...`

## 2.4 Outils collaboratifs utilisés  
- Éditeur Markdown partagé (HedgeDoc / HackMD)  
- Communication (Discord, Teams, etc.)  
- Tableaux de suivi  

---

# 3. 🛠️ Installation et configuration des solutions étudiées

> [!NOTE]
> Cette partie représente le gros du travail technique : installation, captures, schémas, configurations, difficultés rencontrées.

## 3.1 Présentation des technologies choisies  

### 3.1.1 Solution A  
- Principe  
- Architecture  
- Avantages / inconvénients techniques  

### 3.1.2 Solution B  
- Principe  
- Architecture  
- Avantages / inconvénients techniques  

---

## 3.2 Matériel utilisé (infrastructure du rack B202)

| Serveur | Modèle  | CPU | RAM | Stockage | Rôle |
|--------|---------|-----|-----|----------|------|
| Srv-01 | Dell XX | …   | …   | SSD/HDD  | …    |
| Srv-02 | …       | …   | …   | …        | …    |
| Srv-03 | …       | …   | …   | …        | …    |

> [!IMPORTANT]
> Une section spéciale doit traiter du stockage distribué (CEPH / Corosync / VSAN).

---

## 3.3 Installation de la solution A

### 3.3.1 Préparation des serveurs  
- Configuration BIOS  
- Configuration réseau  
- RAID si applicable  

### 3.3.2 Installation pas à pas  
Inclure ici :  
- Commandes  
- Copies d’écran  
- Explications techniques  

### 3.3.3 Configuration réseau  

```mermaid
graph LR
  Client --> Node1
  Client --> Node2
  Node1 --- Storage
  Node2 --- Storage
