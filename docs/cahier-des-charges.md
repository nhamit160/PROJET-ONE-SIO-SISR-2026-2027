# CAHIER DES CHARGES — INFRASTRUCTURE IT, RÉSEAU & CYBERSÉCURITÉ

**Client :** FA SAS (FrameAxis SAS) | **Prestataire :** Équipe IBN | **Version :** 1.0 (Document de travail)

---

## Fiche synthétique et cadrage du projet

| Rubrique | Détail |
| :--- | :--- |
| **Intitulé du projet** | Conception et déploiement de l'infrastructure informatique, réseau et cybersécurité |
| **Client** | FA SAS (FrameAxis SAS) |
| **Représentant client** | Linus TORVALDS (Directeur des Systèmes d'Information / CTO) |
| **Prestataire / Équipe** | Équipe IBN |
| **Co-chefs de projet** | Bruno N'KOMBO & Nils HAMITOUCHE |
| **Référent technique & Sécurité** | Nils HAMITOUCHE |
| **Co-ref. Systèmes & Virtualisation** | Bruno N'KOMBO & Idir SAID LHADJ |
| **Co-ref. Services & IPBX** | Lithicia METAHRI & Lyana RAMANANARIVO |
| **Co-ref. Documentation & Recette** | Gabriel KOUASI OUIA & Lyana RAMANANARIVO |
| **Périmètre global** | 3 bâtiments interconnectés par fibre optique, 120 utilisateurs (40/bâtiment), Cœur L3 redondé, Sécurité périmétrique Fortinet, Wi-Fi 6, Virtualisation Proxmox, IPBX & Sauvegardes |
| **Version & Date** | Version 1.0 (Document de travail) — 9 novembre 2026 |
| **Statut du document** | Prévisionnel à valider avec le client et les devis fournisseurs |

> *Document de cadrage fonctionnel et technique. Il définit les besoins, les contraintes, les exigences et les critères de recette. Les configurations détaillées des équipements seront décrites dans le futur DAT (Document d'Architecture Technique). Les références et montants budgétaires constituent des hypothèses de travail et non des devis fermes.*

---

## Gestion du document et validations attendues

### Historique des versions

| Version | Date | Auteur | Modifications |
| :---: | :---: | :--- | :--- |
| **0.1** | 09/11/2026 | Équipe IBN | Création de la trame et première estimation budgétaire. |
| **1.0** | 09/11/2026 | Équipe IBN | Consolidation du périmètre, du dimensionnement matériel et validation de l'équipe. |

### Validations attendues

| Rôle | Nom / Fonction | Date | Visa / Statut |
| :--- | :--- | :---: | :---: |
| **Représentant FA SAS** | Linus TORVALDS (Directeur des SI / CTO) | En attente | À signer |
| **Co-chefs de projet IBN** | Bruno N'KOMBO & Nils HAMITOUCHE | 09/11/2026 | Validé |
| **Référent technique** | Nils HAMITOUCHE | 09/11/2026 | Validé |

---

## Sommaire

1. [Présentation et contexte du projet](#1-présentation-et-contexte-du-projet)
2. [Périmètre et objectifs](#2-périmètre-et-objectifs)
3. [Analyse de l'existant](#3-analyse-de-lexistant)
4. [Expression des besoins](#4-expression-des-besoins)
5. [Exigences fonctionnelles et techniques](#5-exigences-fonctionnelles-et-techniques)
6. [Architecture cible et principes de conception](#6-architecture-cible-et-principes-de-conception)
7. [Sécurité et continuité de service](#7-sécurité-et-continuité-de-service)
8. [Dimensionnement matériel](#8-dimensionnement-matériel)
9. [Logiciels, services et exploitation](#9-logiciels-services-et-exploitation)
10. [Budget prévisionnel et coût total de possession](#10-budget-prévisionnel-et-coût-total-de-possession)
11. [Organisation et planification](#11-organisation-et-planification)
12. [Tests, recette et critères d'acceptation](#12-tests-recette-et-critères-dacceptation)
13. [Livrables et documentation](#13-livrables-et-documentation)
14. [Risques, hypothèses et points à arbitrer](#14-risques-hypothèses-et-points-à-arbitrer)
15. [Transition vers le DAT](#15-transition-vers-le-dat)
16. [Annexes et glossaire](#16-annexes-et-glossaire)

---

## 1. Présentation et contexte du projet

### 1.1 Contexte général
**FrameAxis SAS (FA SAS)** est une entreprise nécessitant la conception d'une infrastructure informatique cohérente, hautement disponible, sécurisée, administrable et évolutive pour trois bâtiments interconnectés (**Bâtiments A, B et C**). L'ingénierie et le déploiement de cette infrastructure sont confiés à l'équipe projet **IBN**.

L'infrastructure cible doit accueillir environ **120 utilisateurs**, répartis de manière équilibrée entre les trois bâtiments, soit une hypothèse de travail de **40 utilisateurs par bâtiment**.

La maquette de référence Packet Tracer constitue la base d'analyse du réseau. Elle décrit une topologie en étoile structurée autour du **Bâtiment A**, lequel héberge la salle serveur centralisée, l'IPBX et la sortie Internet principale. La commutation d'accès est assurée par deux switches d'accès par bâtiment, chacun relié aux deux switches de cœur de réseau. La haute disponibilité et la répartition de charge HSRP sont prévues entre les deux switches de cœur `CORE-1` et `CORE-2`.

### 1.2 Problématique
> *Comment fournir aux 120 utilisateurs de FrameAxis SAS un accès fiable et performant aux applications et ressources de l'entreprise, tout en assurant la segmentation hermétique des usages, la protection contre les cybermenaces, la continuité de service et la maîtrise complète des coûts d'investissement et d'exploitation ?*

### 1.3 Objectifs généraux du projet
* **Moderniser et structurer** le réseau LAN/WAN interconnectant les trois bâtiments.
* **Éliminer les points uniques de défaillance** (SPOF) sur l'ensemble des composants critiques (Cœur L3, pare-feu périmétrique, liens montants).
* **Protéger et filtrer** l'ensemble des flux entrants, sortants et inter-VLAN selon le principe du moindre privilège.
* **Centraliser** l'administration système, la supervision réseau (alerting/métriques) et la journalisation des événements.
* **Garantir l'évolutivité** de l'infrastructure afin d'absorber la croissance future du nombre d'utilisateurs et des services applicatifs.
* **Fournir une documentation exhaustive** (DAT, schémas physiques/logiques, dossier d'exploitation) garantissant une exploitation et une maintenance maîtrisées.

---

## 2. Périmètre et objectifs

### 2.1 Périmètre inclus
Le projet englobe la totalité des prestations techniques et matérielles suivantes :
* Réseau LAN des trois bâtiments et l'interconnexion par liaison fibre optique OM4.
* Équipements de cœur de réseau niveau 3 (L3), commutation d'accès PoE+ et segmentation par VLANs.
* Accès Internet sécurisé via deux opérateurs distants (FAI 1 et FAI 2) et sécurité périmétrique assurée par une paire de pare-feux Fortinet en Haute Disponibilité.
* Réseaux Wi-Fi professionnel interne et Wi-Fi invité totalement isolé.
* Infrastructure de deux serveurs physiques (Dell R360), hyperviseur de virtualisation (Proxmox VE), stockage centralisé (NAS Synology RS2423+) et stratégie de sauvegarde 3-2-1.
* Architecture de Téléphonie IP / IPBX, plateforme de supervision centralisée (Zabbix/PRTG) et administration réseau sécurisée.
* Travaux d'installation, pré-configuration, intégration, recettes techniques et fourniture de la documentation complète.

### 2.2 Hors périmètre (à confirmer par FA SAS)
Les éléments suivants ne font pas partie des livrables de l'équipe IBN :
* Achat ou renouvellement du parc des postes de travail utilisateurs (PC portables/fixes).
* Écrans, imprimantes et périphériques individuels (hors raccordement et attribution d'IP sur le réseau).
* Travaux de génie civil, création de conduits inter-bâtiments et réfection électrique lourde des locaux.
* Souscription des abonnements téléphoniques détaillés et coûts des communications FAI.
* Migration approfondie des données et applications métier non décrites explicitement dans le présent document.

### 2.3 Objectifs mesurables et méthodes de vérification

| Objectif | Cible de cadrage | Méthode de vérification |
| :--- | :--- | :--- |
| **Capacité réseau** | Support de 120 utilisateurs au démarrage + $30\%$ de marge | Inventaire physique, tables d'adresses MAC et tests de charge. |
| **Accès aux services** | Connectivité fluide des VLANs autorisés vers les serveurs | Tests ICMP (`ping`), requêtes DNS, attribution DHCP et accès applicatifs. |
| **Redondance LAN** | Basculement d'un commutateur de cœur sans perte de service | Test de déconnexion physique du cœur actif `CORE-1` sous trafic. |
| **Redondance Pare-feu** | Basculement transparent du cluster FortiGate HA | Simulation de panne électrique du nœud actif FortiGate. |
| **Wi-Fi Invités** | Accès Internet strict sans aucun accès au LAN privé | Scans de ports et tentatives de `ping` vers l'adressage RFC 1918 interne. |
| **Sauvegardes IT** | Sauvegardes planifiées, chiffrées et restaurables | Test réel de restauration d'une VM et d'un jeu de données sur NAS. |

---

## 3. Analyse de l'existant

### 3.1 Synthèse de la maquette de référence
Le schéma de référence Packet Tracer disponible sur GitHub présente les caractéristiques architecturales suivantes :
* **Topologie :** Trois bâtiments (A, B, C) structurés en étoile autour du Bâtiment A.
* **Cœur de réseau :** Deux commutateurs L3 (`CORE-1` et `CORE-2`) reliés par un EtherChannel LACP (`Po1`).
* **Redondance passerelle :** Protocole HSRP assurant une passerelle virtuelle par VLAN, avec répartition de charge des VLANs actifs entre `CORE-1` et `CORE-2`.
* **Commutation d'accès :** Six switches L2 PoE+ prévus pour alimenter les postes, téléphones IP et bornes Wi-Fi.
* **Liaisons :** Liaisons inter-bâtiments en Fibre Optique OM4 ; câblage intérieur en cuivre Catégorie 6 / 6A.
* **Sécurité initiale de la maquette :** Deux FAI, deux routeurs de bordure et un pare-feu Cisco ASA unique.
* **Administration :** Accès SSH sur les cœurs autorisé uniquement à partir du VLAN 10 Administration.

### 3.2 Plan d'adressage IP et segmentation VLAN de référence

| VLAN | Nom du segment | Plage IP / Masque | Passerelle virtuelle (VIP) / Remarque |
| :---: | :--- | :--- | :--- |
| **10** | `ADMINISTRATION` | `192.168.10.0/27` | `192.168.10.30` (Plage d'administration dédiée) |
| **20** | `UTILISATEURS` | `192.168.20.0/24` | `192.168.20.254` (Postes de travail clients) |
| **30** | `VOIX` | `192.168.30.0/24` | `192.168.30.254` (Téléphonie IP / IP Phones) |
| **40** | `SERVEURS` | `192.168.40.0/26` | `192.168.40.62` (Salle serveur Bâtiment A) |
| **50** | `IMPRIMANTES` | `192.168.50.0/27` | `192.168.50.30` (Équipements d'impression) |
| **60** | `WIFI_INTERNE` | `192.168.60.0/23` | `192.168.61.254` (Bornes Wi-Fi internes) |
| **70** | `WIFI_INVITES` | `192.168.70.0/24` | `192.168.70.254` (Accès invités isolé) |
| **99** | `DMZ` | `192.168.99.0/28` | `192.168.99.14` (Présent côté pare-feu) |
| **100** | `TRANSIT_FW` | `192.168.100.0/29` | `192.168.100.6` (Transit entre Cœur et Pare-feu) |

### 3.3 Points d'attention et ajustements requis
* **VLAN 60 en `/23` :** La plage s'étend de `192.168.60.0` à `192.168.61.255`. La passerelle `192.168.61.254` appartient bien au sous-réseau. Aucun VLAN 61 séparé ne doit être créé.
* **DMZ (VLAN 99) :** Le segment DMZ est directement rattaché au pare-feu. Le routage, les interfaces et les politiques de filtrage strictes doivent être validés dans le DAT.
* **Évolution Pare-feu :** La maquette de départ repose sur un Cisco ASA unique ne fournissant pas de Haute Disponibilité. L'architecture cible remplace cet équipement par un cluster de deux pare-feux Fortinet FortiGate 120G en HA Active-Passive.

---

## 4. Expression des besoins

### 4.1 Besoins des Utilisateurs
* Disposer d'un accès réseau filaire stable et performant (Gigabit) sur leur poste de travail.
* Bénéficier d'une couverture Wi-Fi professionnelle fluide dans les trois bâtiments.
* Passer des appels via la Téléphonie IP sans latence, hachage ni dégradation de la voix (priorisation QoS).
* Accéder de manière sécurisée aux applications internes et à Internet conformément à leurs droits.

### 4.2 Besoins de l'Équipe IT
* Administrer l'ensemble des commutateurs, pare-feux et serveurs via des canaux chiffrés sécurisés (SSH/HTTPS) depuis un VLAN réservé (VLAN 10).
* Centraliser la supervision globale du réseau via des journaux (`syslog`), des métriques d'état et un tableau de bord d'alerte (Zabbix/PRTG).
* Sauvegarder automatiquement les configurations des équipements et les données des machines virtuelles.
* Pouvoir isoler rapidement un hôte, un VLAN ou un flux suspect en cas d'incident de sécurité.

### 4.3 Besoins de la Direction
* Maîtriser l'investissement initial (CAPEX) et les coûts récurrents de maintenance/licences (OPEX).
* Limiter au maximum le risque d'interruption d'activité des services essentiels de l'entreprise.
* Conserver une capacité d'extension du réseau sans nécessiter de refonte architecturale majeure.
* Disposer d'un dossier de documentation complet et d'un inventaire physique à jour.

---

## 5. Exigences fonctionnelles et techniques

Les priorités d'exigences sont définies comme suit : **$P1$** = Indispensable (bloquant pour la recette), **$P2$** = Important, **$P3$** = Souhaitable.

| ID | Libellé de l'exigence | Prio. | Critère de vérification |
| :---: | :--- | :---: | :--- |
| **F-01** | Les 120 utilisateurs doivent accéder aux services autorisés. | $P1$ | Tests de connectivité par VLAN applicatif. |
| **F-02** | Les 3 bâtiments doivent être interconnectés par fibre optique. | $P1$ | Contrôle de l'état des liens et tests de débit. |
| **F-03** | Le réseau doit être segmenté hermétiquement par VLAN. | $P1$ | Audit des tables 802.1Q et des liens trunks. |
| **F-04** | Le Wi-Fi invité doit être totalement isolé du LAN privé. | $P1$ | Tentative d'accès aux adresses IP privées bloquée. |
| **F-05** | Adressage automatique dynamique par serveur DHCP. | $P1$ | Vérification des baux DHCP, passerelles et DNS. |
| **F-06** | Hébergement des services internes sur serveurs prévus. | $P1$ | Validation DNS, Active Directory et applicatifs. |
| **T-01** | Le cœur de réseau doit disposer de deux commutateurs L3. | $P1$ | Inventaire matériel et validation des schémas. |
| **T-02** | Redondance des passerelles réseau via le protocole HSRP. | $P1$ | Test de coupure d'un cœur et basculement. |
| **T-03** | Commutateurs d'accès PoE+ pour alimenter bornes et IPBX. | $P1$ | Test d'alimentation électrique et budget PoE+. |
| **T-04** | Liens montants (uplinks) à haute capacité (cible $10\text{ Gb/s}$). | $P2$ | Fiches techniques et tests de charge de liaison. |
| **S-01** | Filtrage strict des flux inter-VLAN par pare-feu / ACL. | $P1$ | Tests de validation d'accès autorisés/interdits. |
| **S-02** | Administration sécurisée restreinte au VLAN 10 dédié. | $P1$ | Test d'accès SSH/HTTPS refusé depuis autres VLANs. |
| **S-03** | Accès distants sécurisés par tunnel VPN chiffré. | $P1$ | Test d'authentification et de communication VPN. |
| **E-01** | Sauvegardes planifiées, chiffrées et restaurables (3-2-1). | $P1$ | Test réel de restauration d'un jeu de données. |
| **E-02** | Supervision globale et remontée d'alertes en temps réel. | $P2$ | Déclenchement d'une alerte lors d'une déconnexion. |

---

## 6. Architecture cible et principes de conception

### 6.1 Architecture logique ciblée
L'architecture cible est organisée en couches hiérarchiques redondées :
1. **Zone d'Accès Internet & Bordure :** Deux liaisons opérateurs distantes (FAI 1 et FAI 2) raccordées à deux pare-feux Fortinet FortiGate 120G configurés en cluster Haute Disponibilité Active-Passive.
2. **Cœur de Réseau (Niveau 3) :** Deux commutateurs de cœur Cisco Catalyst 9300 interconnectés par EtherChannel LACP. Ils assurent le routage inter-VLAN, la redondance HSRP et le filtrage principal.
3. **Commutation d'Accès (Niveau 2) :** Six commutateurs d'accès Cisco Catalyst 9200L PoE+ distribués dans les 3 bâtiments (2 switches par bâtiment), apportant la connectivité physique aux postes, téléphones IP et bornes Wi-Fi.
4. **Salle Serveur (Bâtiment A) :** Hébergement centralisé des 2 serveurs physiques Dell PowerEdge R360, de la baie de stockage NAS Synology RS2423+, de l'IPBX et du contrôleur Wi-Fi.

### 6.2 Répartition physique et capacitative par bâtiment

| Zone / Bâtiment | Utilisateurs cibles | Switches d'accès | Ports installés | Marge disponible |
| :--- | :---: | :---: | :---: | :---: |
| **Bâtiment A (Principal)** | 40 | $2 \times 48$ ports PoE+ | 96 ports | $+140\%$ |
| **Bâtiment B** | 40 | $2 \times 48$ ports PoE+ | 96 ports | $+140\%$ |
| **Bâtiment C** | 40 | $2 \times 48$ ports PoE+ | 96 ports | $+140\%$ |
| **TOTAL GÉNÉRAL** | **120** | **$6 \times 48$ ports** | **288 ports** | **Marge capacitative haute** |

### 6.3 Principes directeurs de conception
* **Doublement du Cœur :** Utilisation de deux équipements L3 indépendants pour éliminer tout risque d'interruption du cœur de réseau.
* **Redondance HSRP :** Passerelle virtuelle unique par VLAN. Répartition active de la charge : `CORE-1` est passerelle active pour les VLANs 10, 30, 50, 70, 100 ; `CORE-2` pour les VLANs 20, 40, 60.
* **Prévention des boucles :** Déploiement du protocole Rapid-PVST+ aligné avec les instances HSRP.
* **Liaisons montantes redondantes :** Chaque commutateur d'accès est raccordé par double liaison fibre vers `CORE-1` et `CORE-2`.
* **Support physique :** Liaisons inter-bâtiments en Fibre Optique OM4 ; câblage horizontal intérieur en cuivre Catégorie 6A.

---

## 7. Sécurité et continuité de service

### 7.1 Sécurité périmétrique et UTM
La protection du périmètre repose sur deux pare-feux **Fortinet FortiGate 120G** configurés en cluster Haute Disponibilité Active-Passive. Les fonctionnalités activées comprennent :
* Filtrage d'état (Stateful Inspection) selon le principe du moindre privilège.
* Système de prévention des intrusions (IPS), Antivirus d'interflux et Filtrage Web/DNS.
* Contrôle applicatif pour réguler les flux d'entreprise et bloquer les applications non autorisées.
* Tunnels VPN IPsec inter-sites et VPN SSL pour les accès nomades sécurisés.

### 7.2 Segmentation et contrôle des flux inter-VLAN
* **VLAN 10 (Administration) :** Accessible exclusivement depuis des postes d'administration identifiés. Accès SSH/HTTPS strictement filtré par ACL.
* **VLAN 70 (Wi-Fi Invités) :** Directement acheminé vers le pare-feu, sans aucun routage possible vers les sous-réseaux internes d'entreprise.
* **VLAN 30 (Voix) :** Flux réservés aux communications VoIP entre les téléphones IP et l'IPBX. Priorisation QoS appliquée.
* **VLAN 99 (DMZ) :** Isolé du réseau LAN interne, hébergeant uniquement les services publics autorisés et surveillés.

### 7.3 Continuité d'activité et stratégie de sauvegarde
* Validation rigoureuse des scénarios de basculement HA du pare-feu et HSRP des cœurs réseau.
* Sauvegarde automatique hebdomadaire de la configuration de l'ensemble des commutateurs et pare-feux.
* Mise en œuvre d'une **stratégie de sauvegarde 3-2-1** : 3 copies des données, sur 2 supports différents (NAS local Synology RS2423+), dont 1 copie externalisée et chiffrée.

### 7.4 Conformité et données personnelles
Les traitements de données personnelles doivent respecter les exigences du RGPD et les directives internes de FA SAS. Les règles de rétention des journaux d'accès, la gestion des privilèges et la protection des sauvegardes seront formalisées dans le dossier d'exploitation.

---

## 8. Dimensionnement matériel

Les montants ci-dessous constituent la base de dimensionnement prévisionnelle du projet :

| Famille / Référence indicative | Qté | Budget unitaire TTC | Total TTC |
| :--- | :---: | :---: | :---: |
| Cisco Catalyst C9200L-48P-4X-E (48 ports PoE+) | 6 | 3 982 € | 23 890 € |
| Cisco Catalyst 9300 L3 (Référence à confirmer) | 2 | 3 808 € | 7 616 € |
| Modules optiques SFP+ / Interconnexion cœur | 1 lot | — | 1 500 € |
| Fortinet FortiGate 120G | 2 | 2 850 € | 5 700 € |
| Bundle UTP + FortiCare (Hypothèse 1 an) | 2 | 3 127 € | 6 255 € |
| Dell PowerEdge R360 (128 Go RAM ECC cible) | 2 | 7 000 € | 14 000 € |
| Synology RS2423+ avec lot de disques durs | 1 lot | — | 4 532 € |
| Bornes Wi-Fi 6 + accessoires de fixation | 9 | — | 6 620 € |
| Baies, câblage Cat 6A, fibre OM4, Onduleurs, PDU | 1 lot | — | 9 900 € |
| Téléphonie IP / IPBX (Provision de départ) | 1 lot | — | 2 000 € |

### Justification du dimensionnement
* **Commutation :** 6 switches de 48 ports (288 ports) offrent une réserve importante par rapport aux 120 utilisateurs, permettant de raccorder sereinement les téléphones IP, imprimantes et bornes Wi-Fi.
* **Wi-Fi :** La provision de 9 bornes Wi-Fi 6 (3 bornes par bâtiment) nécessite une étude de couverture radio sur site.
* **Serveurs physiques :** Les 2 serveurs Dell PowerEdge R360 hébergeront l'ensemble des machines virtuelles sous hyperviseur Proxmox VE.

---

## 9. Logiciels, services et exploitation

| Service IT | Solution logicielle retenue | Rôle et fonction |
| :--- | :--- | :--- |
| **Virtualisation** | Proxmox VE | Hébergement et haute disponibilité des serveurs virtuels. |
| **Annuaire central** | Active Directory / Samba4 | Gestion des identités, comptes utilisateurs et Group Policies. |
| **DNS / DHCP** | Services Windows Server / Linux | Résolution de noms interne et attribution dynamique des IPs. |
| **Fichiers** | Serveur de fichiers dédié | Partage des données d'entreprise avec gestion fine des droits. |
| **Supervision** | Zabbix / PRTG | Collecte des métriques, état des liens, alertes et tableaux de bord. |
| **Sauvegarde** | Solution Proxmox / NAS Synology | Protection des machines virtuelles, données et réplication 3-2-1. |
| **Téléphonie IP** | Asterisk / 3CX (à arbitrer) | Gestion des appels IPBX, téléphones IP et enregistrements. |
| **Pare-feu** | FortiOS | Sécurité périmétrique, filtrage UTM, VPN et journalisation. |

### Exploitation courante et maintenance
* Tenue à jour rigoureuse de l'inventaire matériel, des numéros de série et des dates de garantie.
* Exécution et contrôle systématique du succès des tâches de sauvegarde.
* Mises à jour planifiées des équipements et hyperviseurs lors de fenêtres de maintenance convenues.
* Revue trimestrielle des comptes administrateurs et des règles de filtrage pare-feu.

---

## 10. Budget prévisionnel et coût total de possession

### 10.1 Budget de cadrage consolidé (CAPEX)

| Poste de dépense | Montant estimatif TTC |
| :--- | :---: |
| Commutateurs d'accès Cisco Catalyst 9200L (6 unités) | 23 890 € |
| Cœur de réseau Cisco Catalyst 9300 (2 unités) + Optiques | 9 116 € |
| Fortinet FortiGate 120G (2 unités) + Protection UTP (Année 1) | 11 955 € |
| Serveurs physiques Dell PowerEdge R360 (2 unités) | 14 000 € |
| NAS Synology RS2423+ et lot de disques durs | 4 532 € |
| Bornes Wi-Fi 6 professionnelles (9 unités) | 6 620 € |
| Baies, câblage cuivre Cat 6A, fibre OM4, Onduleurs & PDU | 9 900 € |
| Téléphonie IP / IPBX (Provision) | 2 000 € |
| Prestation d'installation, configuration et intégration (Provision) | 8 000 € |
| **SOUS-TOTAL DE TRAVAIL** | **100 013 €** |
| Provision pour imprévus ($10\%$) | 10 001 € |
| **BUDGET INITIAL TOTAL ARRONDI** | **110 014 € TTC** |

### 10.2 Coûts d'exploitation annuels (OPEX) et TCO 3 ans
* **Coûts annuels récurrents :** Estimés entre **14 255 € et 20 055 € TTC / an** (couvrant 2 accès FAI pro, le renouvellement des licences HA Fortinet, l'électricité IT, la sauvegarde externalisée et les contrats de maintenance).
* **Coût Total de Possession (TCO sur 3 ans) :** Évalué dans une fourchette indicative comprise entre **152 800 € et 170 200 € TTC**.

---

## 11. Organisation et planification

### 11.1 Phases du projet
Le projet se déroule en 8 phases successives :
1. **Cadrage** $\rightarrow$ 2. **Conception (DAT)** $\rightarrow$ 3. **Approvisionnement** $\rightarrow$ 4. **Préparation en atelier** $\rightarrow$ 5. **Déploiement sur site** $\rightarrow$ 6. **Migration** $\rightarrow$ 7. **Recette technique** $\rightarrow$ 8. **Transfert d'exploitation**.

### 11.2 Planning et Délais
**Non spécifié dans les éléments fournis.** *(Les dates de démarrage, jalons et planning détaillé seront établis en concertation avec FA SAS lors de la validation du cahier des charges).*

### 11.3 Répartition des rôles et responsabilités (Équipe IBN)

| Nom du membre | Rôle assigné dans l'équipe IBN | Responsabilités principales |
| :--- | :--- | :--- |
| **Bruno N'KOMBO** | Co-chef de projet, Choix matériel & Co-ref. Systèmes & Virtualisation | Coordination globale du projet, sélection du matériel adapté à l'architecture, co-conception Proxmox VE, suivi des risques et validation des livrables. |
| **Nils HAMITOUCHE** | Co-chef de projet, Référent technique en chef & Sécurité | Conception de l'architecture globale, switching L3/L2, redondance HSRP, cluster FortiGate HA, filtrage inter-VLAN et VPN. |
| **Lithicia METAHRI** | Référente Services Network & IPBX | Architecture des services réseau (DNS/DHCP), téléphonie IP (IPBX) et plateforme de supervision centralisée (Zabbix/PRTG). |
| **Lyana RAMANANARIVO** | Co-référente Services, IPBX, Documentation & Recette | Appui aux services réseau/VoIP, co-élaboration des schémas d'architecture, dossier d'exploitation et déroulement du plan de tests. |
| **Idir SAID LHADJ** | Co-référent en chef Systèmes & Virtualisation | Déploiement de l'hyperviseur Proxmox VE, Active Directory, serveurs physiques Dell R360 et stockage NAS Synology. |
| **Gabriel KOUASI OUIA** | Référent Documentation & Recette | Élaboration des schémas réseau, constitution du dossier d'exploitation, exécution du plan de tests et rédaction du PV de recette. |

---

## 12. Tests, recette et critères d'acceptation

Chaque test sera consigné dans le PV de recette avec son statut, la preuve associée (capture/journal) et le nom du testeur :

| ID | Intitulé du test | Résultat attendu et critère de succès |
| :---: | :--- | :--- |
| **R-01** | Obtention DHCP par VLAN | Adresse IP, masque, passerelle et DNS attribués conformément au plan. |
| **R-02** | Résolution DNS interne/externe | Noms de domaine résolus correctement selon les zones autorisées. |
| **R-03** | Communication inter-VLAN | Les seuls flux nécessaires et autorisés par le pare-feu traversent. |
| **R-04** | Isolation du Wi-Fi invité | Blocage strict de tout accès vers les adresses IP privées du LAN. |
| **R-05** | Panne du commutateur `CORE-1` | Les passerelles basculent de manière transparente sur `CORE-2` via HSRP. |
| **R-06** | Panne du FortiGate actif | Le membre secondaire prend le rôle actif en cluster HA sans coupure. |
| **R-07** | Perte d'un accès FAI | Basculement automatique du trafic WAN vers le second accès opérateur. |
| **R-08** | Panne d'une liaison fibre inter-bâtiment | Maintien de la connectivité via le chemin redondant. |
| **R-09** | Accès SSH d'administration | Accès autorisé uniquement depuis les postes d'administration du VLAN 10. |
| **R-10** | Connexion VPN distant | Authentification réussie et accès strictement limité aux ressources permises. |
| **R-11** | Sauvegarde et restauration | Données et machines virtuelles restaurées et contrôlées avec succès. |
| **R-12** | Supervision d'un équipement en panne | Génération immédiate d'une alerte visible sur le tableau de bord. |

### Critères d'acceptation formels
* $100\%$ des tests classés $P1$ doivent être validés avec succès ou faire l'objet d'une dérogation écrite.
* Aucune règle de filtrage ne doit autoriser d'accès non justifié aux segments sensibles.
* La validité des sauvegardes doit être confirmée par une restauration réelle.

---

## 13. Livrables et documentation

| Livrable | Contenu attendu et description |
| :--- | :--- |
| **Cahier des charges** | Expression formalisée des besoins, périmètre, exigences, budget et recette. |
| **Document d'Architecture Technique (DAT)** | Architecture physique et logique détaillée, plan d'adressage IP, routage et sécurité. |
| **Schéma physique du réseau** | Implantation des baies, serveurs, switches, ports et liaisons fibre/cuivre. |
| **Schéma logique du réseau** | Cartographie des VLANs, sous-réseaux, passerelles virtualisées HSRP et flux. |
| **Inventaire du matériel** | Références exactes, numéros de série, licences et dates de garantie. |
| **Sauvegardes des configurations** | Fichiers de configuration sauvegardés de tous les équipements réseau (hors secrets). |
| **Plan et PV de recette** | Cahier de recette complété, preuves d'exécution et procès-verbal signé. |
| **Dossier d'exploitation** | Guides de procédures : sauvegarde, restauration, mises à jour et gestion d'incidents. |

---

## 14. Risques, hypothèses et points à arbitrer

| Risque ou point ouvert | Conséquence possible | Action recommandée |
| :--- | :--- | :--- |
| **Incertitude sur le nombre exact de prises et terminaux** | Sous/sur-dimensionnement des switches d'accès et du budget PoE+ | Réaliser un inventaire physique précis par bâtiment et par usage. |
| **Absence d'étude Wi-Fi préalable sur site** | Couverture radio insuffisante ou zones d'ombre dans les bâtiments | Effectuer une étude de couverture et de densité radio sur site. |
| **Variabilité des prix du matériel non contractuels** | Dépassement possible du budget prévisionnel | Obtenir 2 à 3 devis fermes auprès des revendeurs matériels. |
| **Dépendance à un NAS local unique pour les sauvegardes** | Vulnérabilité aux pannes physiques ou aux attaques de ransomware | Mettre en place une copie externalisée chiffrée (règle 3-2-1). |
| **Risque de boucle réseau / EtherChannel multi-châssis** | Instabilité du réseau LAN et perte de connectivité globale | Valider la configuration STP, LACP et les fonctionnalités des cœurs. |

---

## 15. Transition vers le DAT

Le Cahier des Charges fixe les exigences. Le **Document d'Architecture Technique (DAT)** décrira les configurations détaillées de mise en œuvre.

### Structure recommandée du futur DAT
1. Introduction et référence $\rightarrow$ 2. Architecture physique $\rightarrow$ 3. Architecture logique $\rightarrow$ 4. Configuration Cœur Cisco (HSRP/STP/Trunks) $\rightarrow$ 5. Configuration Fortinet (HA/UTP/VPN) $\rightarrow$ 6. Infrastructure Serveurs (Proxmox VE/VMs) $\rightarrow$ 7. Réseau Wi-Fi $\rightarrow$ 8. Stratégie de Sauvegarde $\rightarrow$ 9. Supervision Zabbix/PRTG $\rightarrow$ 10. Guide d'Exploitation $\rightarrow$ 11. Cahier de Tests exécuté.

---

## 16. Annexes et glossaire

### 16.1 Référence de la maquette
* **Schéma d'infrastructure Packet Tracer FrameAxis SAS :** [https://nhamit160.github.io/PROJET-ONE-SIO-SISR-2026-2027/schema_infra/Infra_PKT_reseau_FrameAxisSAS.html](https://nhamit160.github.io/PROJET-ONE-SIO-SISR-2026-2027/schema_infra/Infra_PKT_reseau_FrameAxisSAS.html)

### 16.2 Glossaire technique

| Terme | Définition technique |
| :---: | :--- |
| **ACL** | *Access Control List* : Règles autorisant ou refusant des flux réseau. |
| **DAT** | *Document d'Architecture Technique* : Document décrivant la mise en œuvre précise. |
| **DMZ** | *Demilitarized Zone* : Zone réseau isolée destinée aux services exposés. |
| **HA** | *High Availability* : Mécanismes garantissant la haute disponibilité et le basculement. |
| **HSRP** | *Hot Standby Router Protocol* : Protocole Cisco de redondance de passerelle IPv4. |
| **IPS** | *Intrusion Prevention System* : Système de prévention et de blocage des intrusions. |
| **LACP** | *Link Aggregation Control Protocol* : Protocole d'agrégation de liens Ethernet. |
| **NAS** | *Network Attached Storage* : Boîtier de stockage réseau centralisé. |
| **PoE+** | *Power over Ethernet Plus* : Alimentation électrique des équipements via le câble RJ45. |
| **TCO** | *Total Cost of Ownership* : Coût total de possession sur une période donnée. |
| **VLAN** | *Virtual Local Area Network* : Réseau local virtuel permettant de segmenter un réseau. |
| **VPN** | *Virtual Private Network* : Tunnel de communication chiffré et sécurisé. |

---
*Fin du document — Cahier des Charges Infrastructure IT, Réseau & Cybersécurité FrameAxis SAS (FA SAS)*