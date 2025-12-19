



1. Mise en place du serveur Proxmox (pve1)
1.1. Accès au serveur et remise à zéro de l’interface d’administration

Au départ, le serveur physique (qui allait devenir pve1) était déjà installé et câblé sur le réseau, avec ses disques présents, mais nous n’avions pas les bons accès à l’interface d’administration distante.

Nous avons donc commencé par :

    Identifier le serveur sur le réseau (adresse IP  
    d’administration, switch, port).

    Essayer de nous connecter à l’interface de gestion
    distante (type iLO ).

    Constater que les identifiants d’origine n’étaient
    pas valides.

Comme nous avions besoin de la console distante pour booter sur l’ISO Proxmox et surveiller l’installation, nous avons décidé de réinitialiser cette interface :

    Reset de l’interface d’admin (remise à zéro des
    paramètres et du mot de passe).

    Reconfiguration minimale (adresse IP d’admin, masque,
    passerelle) pour y accéder depuis notre poste.

    [ Capture : interface d’administration distante après reset / page de login ]

Une fois l’accès restauré, nous pouvions agir sur le serveur comme si nous étions devant l’écran et le clavier.
1.2. Préparation des disques et choix de l’architecture de stockage

Le serveur disposait déjà de plusieurs disques physiques. L’objectif était de :

    Dédier un disque (ou un groupe) au système Proxmox,

    Garder de l’espace pour stocker ensuite les disques des VM d’infrastructure
    (pve‑vm1, pve‑vm2, pve‑vm3).

Lors de la phase d’installation de Proxmox, nous avons :

    Listé les disques visibles par l’installateur (nom,taille,type).

    Sélectionné le disque principal pour héberger Proxmox VE.

    Choisi un schéma de partitionnement adapté (dans notre cas,
    on a utilisé le stockage type ZFS/LVM tel que proposé par défaut
    pour pouvoir ensuite créer un pool qui servira aux futures VM).

    [ Capture : écran de sélection du disque système dans l’installateur Proxmox ]

Ce choix de stockage est important, car plus tard, c’est sur ce même stockage que nous avons créé les disques des nœuds virtuels (pve‑vm1, pve‑vm2, pve‑vm3) ainsi que le disque OSD pour Ceph.
1.3. Installation de Proxmox VE depuis l’ISO

Une fois l’accès à la console restauré et le disque choisi, nous avons procédé à l’installation de Proxmox :

    Montage de l’ISO Proxmox VE dans la console distante du serveur.

    Redémarrage du serveur en forçant le boot sur l’ISO.

    Sur l’écran d’accueil, sélection de :
    “Install Proxmox VE”.

    [ Capture : écran d’accueil de l’installateur Proxmox VE ]

Lors de cette première tentative d’installation, nous avons rencontré une anomalie importante : l’installateur Proxmox n’affichait aucun des disques physiques du serveur.
Après plusieurs vérifications, nous avons constaté que le problème provenait du RAID matériel encore actif sur le contrôleur du serveur. Les disques étaient regroupés en un volume RAID propriétaire que Proxmox ne pouvait pas exploiter directement.

Pour corriger ce problème, nous avons :

    accédé au BIOS/UEFI du contrôleur RAID,

    supprimé la configuration RAID existante,

    réinitialisé chaque disque en mode JBOD (Just a Bunch Of Disks)
    ou équivalent,

    redémarré l’installateur Proxmox.

    [ Capture : écran du contrôleur RAID / suppression du volume RAID ]

Une fois le RAID désactivé, l’ensemble des disques physiques est apparu correctement dans l’installateur, ce qui nous a permis de poursuivre la procédure normale d’installation.
Cet incident nous a permis de comprendre l’impact du RAID matériel dans une configuration destinée à accueillir des solutions comme Ceph, qui exigent un accès direct aux disques.

Ensuite, nous avons suivi les étapes classiques de l’installateur :

    Acceptation de la licence.

    Sélection du disque cible (celui identifié précédemment).

    Choix du pays, fuseau horaire, layout clavier.

    Définition du mot de passe root et de l’adresse e‑mail d’admin.

    [ Capture : écran de configuration du mot de passe root et de l’e‑mail ]

1.4. Configuration réseau de base du serveur (IP, passerelle, DNS)

Pendant l’installation, nous avons configuré l’adresse IP de gestion de Proxmox, qui sera ensuite utilisée pour accéder à l’interface Web :

    Attribution d’une IP fixe au serveur
    10.202.4.31
    
    Configuration du masque de sous‑réseau.

    Définition de la passerelle (gateway) pour la sortie vers le réseau. 10.202.255.254

    Définition du serveur DNS. 8.8.8.8

    [ Capture : écran de configuration réseau de Proxmox dans l’installateur ]

Cette étape est cruciale, car c’est elle qui permet ensuite d’accéder à l’interface Web via :

https://10.202.4.31:8006

1.5. Premier démarrage de Proxmox et vérification de l’accès Web

Après l’installation :

    Le serveur a redémarré sur le disque système.

    Nous avons vérifié sur la console distante que Proxmox démarrait correctement

    [ Capture : écran de login sur la console Proxmox après installation ]

    Depuis notre poste, nous avons ouvert un navigateur et saisi l’URL :

https://10.202.4.31:8006

    Le navigateur a affiché un avertissement de certificat (certificat auto‑signé),
    que nous avons accepté.

    Nous nous sommes connectés avec le compte :

    Utilisateur : root

    Mot de passe : celui défini à l’installation. (rootroot)

    [ Capture : page de login Web de Proxmox VE ]

Une fois connectés, nous avions accès à :

    l’arborescence Datacenter → pve1,

    la vue des disques du serveur,

    la configuration réseau (bridge par défaut vmbr0),

    l’état général du système (CPU, RAM, stockage).

    [ Capture : tableau de bord Proxmox après première connexion ]

1.6. Vérifications techniques après installation

Avant d’aller plus loin, nous avons effectué quelques vérifications rapides pour s’assurer que le serveur était correctement prêt pour accueillir nos futures VM de cluster :

    Vérification de l’espace disque disponible dans l’interface Proxmox 
    (nœud → Disks / Storage).

    Vérification du réseau :

    Que vmbr0 soit bien associé à l’interface physique,

    Que l’IP répond au ping depuis nos postes.

    Vérification des mises à jour disponibles (manque de licence)

    [ Capture : écran “Disks” ou “Storage” montrant le stockage local disponible ]

À ce stade, le serveur Proxmox “pve1” était opérationnel et prêt à héberger :

    les VM d’infrastructure (pve‑vm1, pve‑vm2, pve‑vm3),

    la configuration du cluster,

    la mise en place de Ceph et de la haute disponibilité.


2. Création des nœuds Proxmox pve2 et pve3

Après l’installation et la configuration du premier serveur Proxmox (pve1), nous avons procédé à la mise en place des deux autres nœuds nécessaires à la constitution du cluster, à savoir pve2 et pve3.
L’objectif de cette étape était de disposer de trois nœuds Proxmox afin de pouvoir activer les fonctionnalités avancées telles que le cluster, la communication Corosync, le stockage distribué Ceph et la haute disponibilité.

Bien que les serveurs pve2 et pve3 soient déployés dans un environnement virtualisé, ils sont considérés dans la suite du projet comme des nœuds Proxmox à part entière, au même titre que pve1.

2.1. Préparation du stockage pour pve2 et pve3
Lors des premières tentatives de création des serveurs pve2 et pve3, nous avons rencontré des problèmes liés à un manque d’espace de stockage.

Initialement, l’espace disque alloué était insuffisant, ce qui a entraîné :

    des erreurs lors de l’upload des fichiers ISO,

    des échecs pendant l’installation de Proxmox,

    l’impossibilité de prévoir l’espace nécessaire pour la mise en place
    ultérieure de Ceph.

Nous avons donc revu la configuration du stockage afin de :

    augmenter la taille des disques alloués à pve2 et pve3,

    anticiper les besoins liés au cluster et au stockage distribué,

    garantir une base stable pour les étapes suivantes.

[ Capture : stockage insuffisant lors des premières tentatives ]

2.2. Installation du nœud Proxmox pve2
Une fois le stockage corrigé, nous avons procédé à l’installation du second nœud, pve2.

Les actions réalisées ont été les suivantes :

    création du serveur pve2 avec des ressources CPU et mémoire adaptées,

    montage de l’ISO officielle de Proxmox VE,

    installation du système sur le disque prévu,

    configuration du réseau avec une adresse IP fixe,

    définition d’un nom d’hôte unique (pve2).

[ Capture : installation de Proxmox sur pve2 ]
[ Capture : configuration réseau de pve2 ]

Après l’installation, nous avons vérifié l’accès à l’interface web de pve2 afin de confirmer le bon fonctionnement du nœud.

[ Capture : interface web de pve2 après installation ]

2.3. Installation du nœud Proxmox pve3
La mise en place du troisième nœud, pve3, a été réalisée en reproduisant exactement la même procédure que pour pve2.

Pour pve3, nous avons :

    utilisé la même configuration matérielle,

    installé Proxmox VE depuis l’ISO,

    configuré une adresse IP distincte,

    défini un nom d’hôte unique (pve3),

    vérifié l’accès à l’interface d’administration.

[ Capture : installation de Proxmox sur pve3 ]
[ Capture : interface web de pve3 ]

Cette homogénéité entre les trois nœuds est essentielle pour assurer la stabilité du cluster Proxmox.

2.4. Problèmes rencontrés et ajustements liés au stockage
Malgré l’augmentation initiale de l’espace disque, nous avons constaté que certaines partitions du serveur hôte restaient limitées, ce qui provoquait encore des erreurs lors de certaines opérations.

Pour corriger cela, nous avons :

    augmenté la taille du disque du serveur,

    étendu manuellement les volumes afin d’exploiter l’ensemble
    de l’espace disponible,

    redémarré les services concernés pour appliquer les modifications.

[ Capture : commandes d’extension des volumes ]
[ Capture : stockage disponible après correction ]

Ces ajustements ont permis de stabiliser définitivement l’environnement et de préparer correctement la mise en place de Ceph.

2.5. État de l’infrastructure à la fin de cette étape
À la fin de cette étape, l’infrastructure était composée de :

    pve1, premier nœud Proxmox opérationnel,

    pve2 et pve3, deux nœuds supplémentaires correctement installés,

    un stockage suffisamment dimensionné pour le cluster et le stockage distribué.

Les trois nœuds Proxmox étaient désormais prêts à être intégrés dans un cluster Proxmox et à utiliser les services de communication Corosync et de stockage distribué Ceph, qui feront l’objet des sections suivantes.



3. Mise en place du cluster Proxmox et configuration de Corosync

Une fois les trois nœuds Proxmox (pve1, pve2 et pve3) installés et opérationnels, l’étape suivante a consisté à les regrouper au sein d’un cluster Proxmox.
La mise en cluster permet aux différents nœuds de fonctionner comme une seule infrastructure et constitue un prérequis indispensable pour l’activation de la haute disponibilité, du stockage distribué Ceph et de la gestion centralisée.

3.1. Principe de fonctionnement du cluster Proxmox et de Corosync
Le cluster Proxmox repose sur Corosync, un service de communication interne chargé de :

    synchroniser l’état des nœuds,

    gérer le quorum,

    détecter les pannes,

    coordonner les actions du cluster (HA, migrations, etc.).

Chaque nœud communique avec les autres via le réseau, ce qui nécessite :

    des adresses IP fixes,

    une connectivité réseau fiable,

    une configuration homogène sur l’ensemble des serveurs.

Dans notre cas, la communication Corosync s’effectue sur le réseau principal, via le bridge réseau configuré lors de l’installation de Proxmox.

3.2. Création du cluster depuis le premier nœud (pve1)
La création du cluster a été initiée depuis le premier nœud, pve1, qui joue le rôle de nœud initial.

Depuis l’interface web de pve1, nous avons :

    accédé à la section Datacenter,

    sélectionné l’option de création d’un cluster,

    défini un nom de cluster,

    validé les paramètres réseau proposés par défaut pour Corosync.

[ Capture : création du cluster Proxmox sur pve1 ]
[ Capture : paramètres Corosync lors de la création du cluster ]

Une fois le cluster créé, pve1 devient le premier membre du cluster et peut accueillir les autres nœuds.

3.3. Ajout du nœud pve2 au cluster
Après la création du cluster sur pve1, nous avons procédé à l’ajout du second nœud, pve2.

La procédure s’est déroulée comme suit :

    génération d’une commande d’adhésion depuis pve1,

    exécution de cette commande sur pve2,

    vérification de la connexion sécurisée entre les deux nœuds.

[ Capture : commande d’ajout du nœud pve2 ]
[ Capture : confirmation de l’ajout de pve2 dans le cluster ]

Une fois l’opération terminée, pve2 apparaissait correctement dans l’arborescence du cluster au niveau du Datacenter.

3.4. Ajout du nœud pve3 au cluster
La même procédure a été appliquée pour le troisième nœud, pve3.

Nous avons :

    utilisé la commande générée depuis pve1,

    rejoint le cluster depuis pve3,

    vérifié l’état de synchronisation entre les trois nœuds.

[ Capture : ajout du nœud pve3 au cluster ]
[ Capture : vue du cluster avec pve1, pve2 et pve3 ]

À ce stade, les trois nœuds Proxmox étaient membres du même cluster.

3.5. Vérification du quorum et de l’état du cluster
Une fois les trois nœuds intégrés, nous avons procédé aux vérifications nécessaires pour valider le bon fonctionnement du cluster :

    contrôle du quorum (majorité des nœuds disponibles),

    vérification de l’état de Corosync,

    confirmation que chaque nœud communiquait correctement avec les autres.

Ces vérifications ont été réalisées :

    depuis l’interface web Proxmox,

    et via des commandes système sur les nœuds.

[ Capture : état du cluster dans l’interface Proxmox ]
[ Capture : statut Corosync ]

Le cluster étant composé de trois nœuds, le quorum était assuré même en cas de perte d’un nœud, ce qui est indispensable pour la stabilité de l’infrastructure.

3.6. État de l’infrastructure après la mise en cluster
À l’issue de cette étape, l’infrastructure disposait :

    d’un cluster Proxmox fonctionnel,

    d’une communication interne assurée par Corosync,

    d’une gestion centralisée depuis l’interface du Datacenter,

    d’une base solide pour la mise en place du stockage distribué Ceph et de la haute disponibilité.

La mise en cluster marque une étape clé du projet, car elle permet désormais de considérer l’infrastructure comme une entité unique.

4. Mise en place du stockage distribué Ceph

Après la création du cluster Proxmox et la mise en place de la communication Corosync entre les nœuds pve1, pve2 et pve3, nous avons procédé à l’installation du stockage distribué Ceph.
Ceph permet de fournir un stockage partagé, tolérant aux pannes, indispensable au fonctionnement de la haute disponibilité (HA) et à la migration des machines virtuelles entre les nœuds du cluster.

4.1. Principe de fonctionnement de Ceph dans Proxmox
Ceph est un système de stockage distribué reposant sur plusieurs composants :

    MON (Monitor) : assure la cohérence de l’état du cluster Ceph,

    MGR (Manager) : fournit les informations de supervision et de gestion,

    OSD (Object Storage Daemon) : gère les disques physiques et 
    le stockage réel des données.

Dans une infrastructure Proxmox, Ceph permet de créer des volumes RBD (RADOS Block Device) utilisables directement comme disques pour les machines virtuelles.
La réplication des données entre les nœuds garantit la continuité de service en cas de panne d’un serveur.

4.2. Préparation des disques pour Ceph
Avant de pouvoir déployer Ceph, il a été nécessaire de s’assurer que chaque nœud disposait d’un disque libre, non utilisé par le système, destiné à accueillir un OSD.

Nous avons donc :

    vérifié la liste des disques disponibles sur chaque nœud,

    identifié les disques non partitionnés,

    confirmé que le RAID matériel était bien désactivé afin de permettre un accès
    direct aux disques (prérequis Ceph).

[ Capture : vue des disques disponibles sur un nœud Proxmox ]

Cette étape est essentielle, car Ceph nécessite un accès direct aux disques pour garantir ses performances et sa fiabilité.

4.3. Installation de Ceph sur les nœuds du cluster
L’installation de Ceph a été réalisée directement depuis l’interface Proxmox, au niveau du Datacenter.

Les actions effectuées ont été :

    installation des paquets Ceph sur chaque nœud (pve1, pve2, pve3),

    création des services MON et MGR,

    vérification du bon déploiement des services sur l’ensemble du cluster.

[ Capture : installation de Ceph depuis l’interface Proxmox ]
[ Capture : état des services MON et MGR ]

À l’issue de cette étape, le cluster Ceph était initialisé, mais aucun stockage n’était encore disponible tant que les OSD n’étaient pas créés.

4.4. Création des OSD sur chaque nœud
Une fois Ceph installé, nous avons procédé à la création des OSD sur chaque nœud du cluster.

Pour chaque serveur, nous avons :

    sélectionné le disque dédié à Ceph,

    créé un OSD associé à ce disque,

    vérifié que l’OSD apparaissait correctement dans l’interface Proxmox.

[ Capture : création d’un OSD sur pve1 ]
[ Capture : liste des OSD sur l’ensemble du cluster ]

Chaque nœud disposant de son propre OSD, les données sont automatiquement répliquées entre les serveurs selon la politique définie par Ceph.

4.5. Création du pool Ceph et intégration dans Proxmox
Une fois les OSD opérationnels, nous avons créé un pool Ceph, qui représente l’espace de stockage utilisable par les machines virtuelles.

Les actions réalisées ont été :

    création d’un pool avec un facteur de réplication adapté,

    intégration du pool Ceph comme stockage dans Proxmox,

    vérification de la disponibilité du stockage pour la création de disques de VM.

[ Capture : création du pool Ceph ]
[ Capture : stockage Ceph visible dans Proxmox ]

À partir de ce moment, Ceph pouvait être utilisé comme stockage principal pour les machines virtuelles du cluster.

4.6. Vérification de l’état et de la santé du cluster Ceph
Avant de poursuivre avec la haute disponibilité, nous avons vérifié l’état général du cluster Ceph :

    statut global (HEALTH_OK),

    état des OSD (up / in),

    cohérence de la réplication des données.

Ces vérifications ont été réalisées :

    depuis l’interface graphique de Proxmox,

    et à l’aide de commandes Ceph en ligne de commande.

[ Capture : état de santé du cluster Ceph ]

Cette étape permet de s’assurer que le stockage distribué est stable et prêt à être utilisé en production.

4.7. État de l’infrastructure après la mise en place de Ceph
À la fin de cette étape, l’infrastructure disposait :

    d’un cluster Proxmox fonctionnel,

    d’un stockage distribué Ceph opérationnel,

    d’un stockage partagé accessible depuis tous les nœuds,

    d’une base indispensable pour activer la haute disponibilité 
    des machines virtuelles.

Le stockage Ceph constitue un élément central de l’infrastructure, car il garantit la résilience des données et permet la migration automatique des VM en cas de panne d’un nœud.

5. Activation et test de la haute disponibilité (HA)

Une fois le cluster Proxmox opérationnel et le stockage distribué Ceph correctement configuré, nous avons pu activer la haute disponibilité (HA).
La HA permet d’assurer la continuité de service : en cas de panne d’un nœud, les machines virtuelles sont automatiquement redémarrées sur un autre nœud du cluster.

Cette fonctionnalité repose sur :

    le cluster Proxmox (Corosync),

    le stockage partagé Ceph,

    le gestionnaire HA intégré à Proxmox.

5.1. Principe de fonctionnement de la HA dans Proxmox
Dans Proxmox, la haute disponibilité est gérée par le HA Manager, qui surveille en permanence :

    l’état des nœuds du cluster,

    la disponibilité du stockage partagé,

    l’état des machines virtuelles déclarées comme critiques.

Lorsqu’un nœud devient indisponible :

    le cluster détecte la perte via Corosync,

    le quorum est maintenu grâce aux autres nœuds,

    les VM marquées en HA sont automatiquement relancées sur un autre nœud disposant des ressources nécessaires.

Ce mécanisme n’est possible que si les disques des VM sont stockés sur un stockage partagé, ce qui justifie l’utilisation de Ceph.

5.2. Vérification des prérequis à l’activation de la HA
Avant d’activer la HA, nous avons vérifié que tous les prérequis étaient remplis :

    les trois nœuds (pve1, pve2, pve3) étaient membres du cluster,

    le quorum était fonctionnel,

    le cluster Ceph était en état HEALTH_OK,

    le stockage Ceph était bien visible sur l’ensemble des nœuds.

[ Capture : état du cluster Proxmox ]
[ Capture : état de santé du cluster Ceph ]



5.3. Activation de la haute disponibilité dans Proxmox

L’activation de la haute disponibilité a été réalisée depuis l’interface web de Proxmox, au niveau du Datacenter.

Nous avons :

    accédé à la section HA,

    vérifié que le service HA était actif,

    contrôlé que chaque nœud était bien pris en compte par le gestionnaire HA.

[ Capture : section HA dans l’interface Proxmox ]

À ce stade, l’infrastructure était prête à accueillir des machines virtuelles en haute disponibilité.

5.4. Ajout d’une machine virtuelle au groupe HA

Pour tester concrètement la haute disponibilité, nous avons sélectionné une machine virtuelle stockée sur Ceph et l’avons ajoutée au système HA.

Les actions réalisées ont été :

    sélection de la VM concernée,

    ajout de la VM dans un groupe HA,

    définition des paramètres de comportement (priorité, redémarrage automatique).

[ Capture : ajout d’une VM dans la configuration HA ]

Une fois la VM ajoutée, elle est désormais surveillée en permanence par le gestionnaire HA.

5.5. Test de bascule en cas de panne d’un nœud

Afin de valider le bon fonctionnement de la haute disponibilité, nous avons effectué un test de panne contrôlé.

La procédure de test a consisté à :

    arrêter volontairement un des nœuds du cluster,

    observer la réaction du cluster,

    vérifier la relance automatique de la VM HA sur un autre nœud.

[ Capture : arrêt du nœud en panne ]
[ Capture : migration/redémarrage automatique de la VM ]

Après quelques instants :

    le nœud défaillant a été détecté comme indisponible,

    la VM a été automatiquement redémarrée sur un autre nœud,

    le service est redevenu accessible sans intervention manuelle.

5.6. Vérification après bascule et retour à la normale
Une fois le test effectué, nous avons :

redémarré le nœud arrêté,

vérifié sa réintégration correcte dans le cluster,

contrôlé l’état du quorum et de la HA,

confirmé que la VM fonctionnait normalement sur son nouveau nœud.

[ Capture : cluster rétabli avec les trois nœuds actifs ]

Ce test a permis de valider le bon fonctionnement du mécanisme de haute disponibilité.

5.7. État de l’infrastructure après activation de la HA
À l’issue de cette étape, l’infrastructure dispose :

    d’un cluster Proxmox stable,

    d’un stockage Ceph partagé et résilient,

    d’une haute disponibilité pleinement fonctionnelle,

    d’une capacité de reprise automatique en cas de panne d’un nœud.


6. Mise en place et test de la migration à chaud

Après l’activation de la haute disponibilité, nous avons mis en place et testé la migration à chaud des machines virtuelles.
La migration à chaud permet de déplacer une machine virtuelle d’un nœud à un autre sans interruption de service, ce qui est essentiel pour la maintenance et l’optimisation des ressources.

Cette fonctionnalité repose sur :

    le cluster Proxmox,

    la communication Corosync,

    le stockage partagé Ceph, qui rend les disques des VM 
    accessibles depuis tous les nœuds.

6.1. Principe de la migration à chaud dans Proxmox

Dans Proxmox, la migration à chaud fonctionne selon le principe suivant :

    la machine virtuelle continue de s’exécuter sur le nœud source,

    l’état de la mémoire est progressivement synchronisé avec le nœud cible,

    une bascule finale est effectuée sans coupure visible pour l’utilisateur.

Ce mécanisme n’est possible que si :

    la VM utilise un stockage partagé (Ceph),

    les nœuds appartiennent au même cluster,

    la connectivité réseau entre les nœuds est stable.

6.2. Vérification des prérequis à la migration

Avant de lancer les tests, nous avons vérifié que :

    la machine virtuelle à migrer était bien stockée sur Ceph,

    les trois nœuds (pve1, pve2, pve3) étaient actifs,

    le cluster et Corosync étaient fonctionnels,

les ressources CPU et RAM étaient disponibles sur les nœuds cibles.

[ Capture : VM utilisant le stockage Ceph ]
[ Capture : état du cluster avant migration ]

6.3. Lancement d’une migration à chaud

Depuis l’interface web Proxmox, nous avons lancé une migration à chaud en :

    sélectionnant la machine virtuelle,

    choisissant l’option Migrer,

    définissant le nœud de destination.

[ Capture : lancement de la migration à chaud ]

Pendant la migration :

    la VM est restée accessible,

    aucun arrêt n’a été nécessaire,

    l’interface Proxmox affichait la progression de l’opération.

6.4. Vérification après migration

Une fois la migration terminée, nous avons vérifié :

    que la VM fonctionnait correctement sur le nouveau nœud,

    que les services hébergés restaient accessibles,

    que l’état du cluster était stable.

[ Capture : VM exécutée sur le nœud cible ]


7. Mise en place du SDN et configuration réseau

Après la validation de la migration à chaud, nous avons travaillé sur la configuration réseau avancée, en utilisant les fonctionnalités SDN (Software Defined Networking) de Proxmox.
L’objectif était de disposer d’un réseau flexible, centralisé et facilement administrable pour les machines virtuelles.

7.1. Principe du SDN dans Proxmox

Le SDN dans Proxmox permet de :

    centraliser la configuration réseau au niveau du Datacenter,

    définir des réseaux logiques indépendants du matériel,

    gérer les bridges, VLAN, NAT et règles réseau de manière cohérente sur tous les nœuds.

Cette approche facilite :

    le déploiement des VM,

    la maintenance du réseau,

    l’évolution de l’infrastructure.

7.2. Réseau par défaut et bridge vmbr0

Lors de l’installation de Proxmox, un bridge réseau par défaut (vmbr0) est créé sur chaque nœud.
Ce bridge relie :

    l’interface réseau physique du serveur,

    les machines virtuelles,

    le réseau local.

Nous avons utilisé ce bridge comme base pour :

    la communication entre les nœuds,

    l’accès réseau des machines virtuelles,

    la communication Corosync et Ceph.

[ Capture : configuration du bridge vmbr0 ]

7.3. Mise en place du SDN au niveau du Datacenter

Depuis l’interface Proxmox, nous avons accédé à la section SDN du Datacenter afin de centraliser la configuration réseau.

Les actions réalisées ont été :

    activation du SDN,

    définition des zones réseau,

    association des bridges existants aux zones SDN.

[ Capture : interface SDN dans Proxmox ]


7.4. Configuration des réseaux pour les machines virtuelles

Dans le cadre du projet, les machines virtuelles ont été connectées :

    soit directement au bridge principal,

    soit à des réseaux définis via le SDN, selon leur rôle.

Cette configuration permet :

    de séparer les flux réseau,

    de préparer des règles de sécurité,

    de faciliter l’évolution future de l’infrastructure.

[ Capture : configuration réseau d’une VM ]

7.5. Vérification du fonctionnement réseau

Pour valider la configuration SDN et réseau, nous avons effectué plusieurs tests :

    communication entre VM,

    accès réseau depuis l’extérieur,

    stabilité du réseau lors des migrations à chaud.

Ces tests ont confirmé que :

    le réseau restait fonctionnel pendant les migrations,

    les VM conservaient leur connectivité,

    la configuration SDN était correctement appliquée sur tous les nœuds.

[ Capture : test de connectivité réseau ]

7.6. État de l’infrastructure après configuration de la migration et du SDN

À la fin de cette phase, l’infrastructure disposait :

    de la migration à chaud fonctionnelle,

    d’un réseau centralisé via le SDN,

    d’une connectivité stable entre les nœuds et les VM,


