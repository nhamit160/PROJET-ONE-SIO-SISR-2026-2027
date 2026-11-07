# Projet One — Infrastructure IT de FrameAxis SAS

> Projet BTS SIO option SISR — **IBN**, prestataire informatique, conçoit l'infrastructure complète de son client **FrameAxis SAS**, une centrale d'achat répartie sur trois bâtiments. Le projet comprend la comparaison de solutions, une maquette Cisco Packet Tracer et la prise en compte des contraintes économiques, techniques, architecturales, réglementaires (RGPD, NIS2) et cyber.

---

## Sommaire

1. [Contexte](#1-contexte)
2. [Objectifs du projet](#2-objectifs-du-projet)
3. [Périmètre](#3-périmètre)
4. [Méthode de comparaison des solutions](#4-méthode-de-comparaison-des-solutions)
5. [Architecture réseau retenue](#5-architecture-réseau-retenue)
6. [Comparatifs à mener](#6-comparatifs-à-mener)
7. [Maquette Cisco Packet Tracer](#7-maquette-cisco-packet-tracer)
8. [Plan d'adressage et VLAN](#8-plan-dadressage-et-vlan)
9. [Haute disponibilité : HSRP et Spanning Tree](#9-haute-disponibilité--hsrp-et-spanning-tree)
10. [Sécurité et conformité](#10-sécurité-et-conformité)
11. [Limites de Packet Tracer](#11-limites-de-packet-tracer)
12. [Organisation du dépôt](#12-organisation-du-dépôt)
13. [Avancement](#13-avancement)

---

## 1. Contexte

### Les acteurs

| Rôle | Société | Responsabilité |
|---|---|---|
| Prestataire | **IBN** | Conception, maquettage, chiffrage et déploiement de l'infrastructure |
| Client | **FrameAxis SAS** | Expression du besoin, validation des choix, exploitation de l'infrastructure livrée |

**Pourquoi distinguer les deux rôles :** chaque livrable (comparatifs, schémas, chiffrage) est rédigé par IBN pour être présenté et validé par FrameAxis SAS. Cela fixe le ton du dossier : une proposition argumentée à un client, pas une simple réalisation technique.

### Le site de FrameAxis SAS

| Élément | Valeur |
|---|---|
| Type d'entreprise | Centrale d'achat |
| Sites | 1 site, **3 bâtiments** (A, B, C) |
| Distance entre bâtiments | Quelques dizaines de mètres |
| Effectif | **40 employés minimum par bâtiment**, soit environ 120 à 150 postes |
| Salle serveur | Bâtiment A |
| Téléphonie | Autocom (PABX) situé dans le bâtiment A |
| Budget | Non limité, mais chaque choix doit être justifié économiquement |

**Pourquoi le bâtiment A est central :** il héberge déjà la salle serveur et l'autocom. Tous les flux critiques y convergent naturellement, donc il devient le cœur de l'infrastructure.

---

## 2. Objectifs du projet

- Concevoir pour FrameAxis SAS une infrastructure **réseau et systèmes** complète, sécurisée et évolutive.
- Pour chaque brique technique, **comparer au minimum 3 solutions** sur plusieurs critères, puis justifier le choix retenu.
- Prendre en compte les dimensions :
  - **techniques** (performances, compatibilité, maintenabilité) ;
  - **économiques** (coût d'acquisition et coût d'exploitation) ;
  - **architecturales** (topologie, redondance, évolutivité) ;
  - **réglementaires** (RGPD, NIS2) ;
  - **cybersécurité** (segmentation, filtrage, authentification, journalisation).
- Prouver la faisabilité technique avec une **maquette Cisco Packet Tracer**, en incluant des tests de bascule.

---

## 3. Périmètre

| Domaine | Contenu |
|---|---|
| Réseau | Topologie, liaisons inter-bâtiments, commutation, routage, VLAN, haute disponibilité, Wi-Fi |
| Sécurité périmétrique | Pare-feu, DMZ, accès Internet redondé |
| Systèmes | Virtualisation, annuaire, services réseau (DHCP, DNS), sauvegarde |
| Téléphonie | Migration de l'autocom vers une téléphonie IP (ToIP) |
| Exploitation | Supervision, journalisation, gestion de parc |
| Conformité | RGPD, NIS2, bonnes pratiques ANSSI |

---

## 4. Méthode de comparaison des solutions

Chaque brique est évaluée avec une **grille pondérée**. On note chaque solution de 1 à 5 sur chaque critère, on multiplie par le poids, puis on additionne.

| Critère | Poids | Ce qu'on évalue |
|---|:---:|---|
| Technique | 25 % | Performances, fonctionnalités, compatibilité avec l'existant |
| Sécurité | 20 % | Fonctions de sécurité, certifications (ANSSI), suivi des vulnérabilités |
| Coût (CAPEX + OPEX) | 15 % | Achat, licences, maintenance sur 5 ans |
| Évolutivité | 15 % | Capacité à absorber la croissance sans tout remplacer |
| Exploitabilité | 15 % | Facilité d'administration, documentation, compétences disponibles |
| Conformité | 10 % | Hébergement des données, souveraineté, aide aux obligations RGPD et NIS2 |

**Pourquoi une grille pondérée :** elle rend la décision traçable et défendable devant un jury. Le budget étant non limité, le coût pèse moins que la technique et la sécurité, sans être ignoré pour autant.

---

## 5. Architecture réseau retenue

**Architecture hiérarchique à deux niveaux (collapsed core), en étoile centrée sur le bâtiment A.**

```
             [FAI 1]          [FAI 2]
                |                |
           [ISR4331-1]      [ISR4331-2]
                 \            /
                  [ASA 5506-X] ---- [DMZ]
                        |
        [CORE-1 3650] ══ Po1 ══ [CORE-2 3650]
         /    |     \           /    |     \
     Bât B  Bât A  Salle serveur  Bât A  Bât C
   (fibre)  (cuivre) (SW-SRV, IPBX, WLC)  (fibre)
```

| Choix | Pourquoi |
|---|---|
| Deux niveaux plutôt que trois | Avec environ 150 utilisateurs, une couche de distribution séparée ajoute du coût et de la complexité sans gain réel |
| Étoile vers le bâtiment A | Serveurs, IPBX et sortie Internet y sont déjà : on centralise le routage et la sécurité au même endroit |
| Deux switchs de cœur | Le cœur est un point de défaillance unique pour toute l'entreprise |
| Chaque switch d'accès relié aux deux cœurs | La perte d'un cœur ou d'une fibre ne coupe aucun bâtiment |
| Fibre OM4 entre bâtiments | Même si quelques dizaines de mètres restent sous la limite de 100 m du cuivre, la fibre **isole électriquement** les bâtiments (terres différentes, risque de foudre) et permet de passer à 10G et plus sans recâbler. Le monomode (OS2) n'est pas nécessaire à cette distance |
| Deux FAI | Une coupure Internet bloquerait les commandes et les échanges avec les fournisseurs, cœur de métier d'une centrale d'achat |
| Migration de l'autocom en ToIP | Plus besoin de câblage téléphonique dédié entre bâtiments, et le réseau cuivre RTC est en cours d'extinction en France |

Le schéma détaillé est disponible dans l'artefact *Infra Centrale d'Achat*.

---

## 6. Comparatifs à mener

Au moins trois solutions par brique. La colonne « Retenu » se remplit une fois la grille de la section 4 appliquée.

| Brique | Solution 1 | Solution 2 | Solution 3 | Retenu |
|---|---|---|---|---|
| Architecture réseau | Collapsed core (2 niveaux) | Hiérarchique 3 niveaux | Réseau plat en cascade | Collapsed core |
| Liaisons inter-bâtiments | Fibre multimode OM4 | Cuivre Cat 6A | Pont radio (faisceau hertzien) | Fibre OM4 |
| Commutation | Cisco Catalyst | Aruba CX (HPE) | Juniper EX | À comparer |
| Pare-feu | Fortinet FortiGate | Stormshield SN (qualifié ANSSI) | OPNsense / pfSense | À comparer |
| Virtualisation | VMware vSphere | Proxmox VE | Microsoft Hyper-V | À comparer |
| Annuaire / identités | Active Directory local | AD + Entra ID (hybride) | Samba AD / FreeIPA | À comparer |
| Téléphonie IP | Cisco CUCM / CME | 3CX | FreePBX (Asterisk) | À comparer |
| Wi-Fi | Cisco (WLC + bornes) | Aruba | Ubiquiti UniFi | À comparer |
| Sauvegarde | Veeam | Proxmox Backup Server | Acronis Cyber Protect | À comparer |
| Supervision | Zabbix | Centreon | PRTG | À comparer |
| Gestion de parc / tickets | GLPI | ServiceNow | Freshservice | À comparer |

**Pourquoi inclure systématiquement une solution libre :** elle sert de référence de coût et montre au jury que l'option open source a été étudiée, même si elle n'est pas retenue.

---

## 7. Maquette Cisco Packet Tracer

### Équipements

| Rôle | Modèle PT | Qté |
|---|---|:---:|
| Cœur L3 | Multilayer Switch **3650-24PS** | 2 |
| Accès (PoE) | Multilayer Switch **3560-24PS** | 6 (2 par bâtiment) |
| Switch salle serveur | Switch **2960-24TT** | 1 |
| Serveurs (DHCP, DNS, AAA, Syslog/NTP) | Server-PT | 4 |
| Serveur web en DMZ | Server-PT | 1 |
| IPBX (autocom) | Router **2811** avec CME | 1 |
| Téléphones | IP Phone **7960** | 1 à 2 par bâtiment |
| Pare-feu | **ASA 5506-X** | 1 |
| Routeurs de bordure | Router **ISR4331** | 2 |
| Simulation FAI | Router 2911 ou Cloud-PT | 2 |
| Contrôleur Wi-Fi | **WLC-3504** | 1 |
| Bornes Wi-Fi | LAP-PT | 1 à 2 par bâtiment |
| Postes | PC-PT, Laptop-PT, Printer-PT | Quelques-uns par bâtiment |

> ⚠️ Le 3650 arrive **sans alimentation** : onglet *Physical*, ajouter le module **AC-POWER-SUPPLY** avant de l'allumer.

**Pourquoi des 3560-24PS en accès :** ils sont PoE, donc les téléphones IP et les bornes s'alimentent par le câble réseau, comme en production.

**Pourquoi quelques postes seulement :** chaque terminal représente un groupe d'environ 40 postes. L'objectif est de prouver que chaque type de flux fonctionne, pas de reproduire 150 machines.

### Ordre de construction

1. Poser et câbler le matériel
2. VLAN et trunks sur tous les switchs
3. Rapid-PVST et EtherChannel
4. SVI, HSRP et `ip routing` sur les cœurs
5. Serveurs, DHCP et `ip helper-address`
6. Téléphonie (CME et `switchport voice vlan`)
7. Wi-Fi (WLC, bornes, SSID)
8. Sortie Internet (routeurs, ASA, NAT, ACL, DMZ)
9. Tests de bascule

---

## 8. Plan d'adressage et VLAN

| VLAN | Nom | Réseau | Passerelle (VIP HSRP) | Porté par |
|:---:|---|---|---|---|
| 10 | Administration | 192.168.10.0/24 | 192.168.10.254 | Cœurs |
| 20 | Utilisateurs | 192.168.20.0/24 | 192.168.20.254 | Cœurs |
| 30 | Voix | 192.168.30.0/24 | 192.168.30.254 | Cœurs |
| 40 | Serveurs | 192.168.40.0/24 | 192.168.40.254 | Cœurs |
| 50 | Imprimantes | 192.168.50.0/24 | 192.168.50.254 | Cœurs |
| 60 | Wi-Fi interne | 192.168.60.0/24 | 192.168.60.254 | Cœurs |
| 70 | Wi-Fi invités | 192.168.70.0/24 | À définir | Cœurs ou ASA |
| 99 | DMZ | 192.168.99.0/24 | 192.168.99.254 | ASA |
| 100 | Transit cœurs ↔ ASA | 192.168.100.0/24 | 192.168.100.254 | Cœurs |

Convention dans chaque VLAN routé : **.252** = CORE-1, **.253** = CORE-2, **.254** = IP virtuelle HSRP.

**Pourquoi un /24 par VLAN :** 254 adresses par VLAN laissent une large marge pour environ 40 postes par bâtiment, et le troisième octet égal au numéro de VLAN rend le plan lisible d'un coup d'œil.

**Pourquoi un VLAN de transit :** l'ASA doit renvoyer le trafic vers une adresse qui survit à la perte d'un cœur. Il pointe donc sa route retour vers la VIP HSRP du VLAN 100, et non vers l'IP réelle d'un cœur.

---

## 9. Haute disponibilité : HSRP et Spanning Tree

Une instance HSRP par VLAN routé (numéro de groupe = numéro de VLAN), avec répartition de charge entre les deux cœurs :

| VLAN | Actif HSRP + root STP | Standby + root secondaire |
|---|---|---|
| 10, 30, 50, 70, 100 | CORE-1 | CORE-2 |
| 20, 40, 60 | CORE-2 | CORE-1 |

**Pourquoi répartir :** sinon un cœur reste inutilisé en dehors des pannes.

**Pourquoi aligner HSRP et root bridge :** si le cœur actif d'un VLAN n'est pas son root bridge, le trafic traverse inutilement l'EtherChannel entre les deux cœurs.

**Pourquoi `preempt` :** il permet au cœur prioritaire de reprendre son rôle à son retour, ce qui préserve la répartition de charge.

Vérification : `show standby brief`, puis un ping continu depuis un PC pendant la coupure d'un cœur.

---

## 10. Sécurité et conformité

### Mesures techniques

| Mesure | Pourquoi |
|---|---|
| Segmentation en VLAN + ACL inter-VLAN | Un poste compromis ne voit pas directement les serveurs |
| VLAN invités isolé | Les visiteurs n'accèdent qu'à Internet |
| DMZ pour les services exposés | Un serveur public compromis n'ouvre pas l'accès au réseau interne |
| Authentification 802.1X / RADIUS | Seuls les équipements et utilisateurs connus accèdent au réseau |
| Administration en SSH uniquement, dans un VLAN dédié | Mots de passe jamais en clair, surface d'administration réduite |
| Port-security et désactivation des ports inutilisés | On empêche le branchement d'un équipement non autorisé |
| Journalisation centralisée (Syslog) et NTP | Traçabilité des incidents avec des horodatages cohérents |
| Sauvegardes selon la règle 3-2-1, avec une copie hors ligne | Résilience face aux rançongiciels |
| MFA sur les accès d'administration et distants | Un mot de passe volé ne suffit plus |

### RGPD

Une centrale d'achat traite des données personnelles : salariés, contacts fournisseurs et clients. L'infrastructure doit permettre :
- la **sécurité des traitements** (article 32 : chiffrement, contrôle d'accès, sauvegarde) ;
- la **traçabilité** des accès aux données ;
- la **détection et la notification des violations** de données à la CNIL sous 72 heures ;
- un **hébergement maîtrisé** des données, à prendre en compte dans le choix des solutions cloud.

### NIS2

NIS2 s'applique aux entités moyennes et grandes des secteurs listés par la directive. Avec 120 à 150 salariés, l'entreprise est une **moyenne entreprise**. Son assujettissement **dépend donc de son secteur d'activité**, par exemple la distribution alimentaire en gros, qui figure dans les secteurs concernés. Ce point est à vérifier et à justifier dans le dossier.

Même sans obligation, les mesures NIS2 servent de **référentiel de bonnes pratiques** : gestion des risques, continuité d'activité, sécurité de la chaîne d'approvisionnement, gestion des incidents, MFA.

### Référentiels

- Guide d'hygiène informatique de l'**ANSSI**
- Recommandations de la **CNIL** sur la sécurité des données personnelles

---

## 11. Limites de Packet Tracer

| Limite | Contournement sur la maquette | En production |
|---|---|---|
| Pas de stacking (StackWise) ni de VSS | HSRP + EtherChannel + Rapid-PVST | Stack ou VSS sur les cœurs |
| Pas de failover ASA | Un seul ASA | Deux pare-feu en actif/passif |
| QoS partielle | Démonstration du VLAN voix | QoS complète (DSCP EF) |
| Pas de modèles de switchs récents | 3650 / 3560 | Gamme Catalyst actuelle |

---

## 12. Organisation du dépôt

```
PROJET-ONE-SIO-SISR-2026-2027/
├── README.md
├── docs/
│   ├── cahier-des-charges.md
│   ├── comparatifs/          # une grille par brique technique
│   ├── schemas/              # schémas réseau (HTML, PNG, PDF)
│   └── conformite/           # analyses RGPD et NIS2
├── packet-tracer/
│   └── FrameAxis_SAS_infra.pkt
└── configs/                  # configurations exportées par équipement
    ├── CORE-1.txt
    ├── CORE-2.txt
    └── ...
```

**Pourquoi exporter les configurations en texte :** un fichier `.pkt` n'est pas lisible sans Packet Tracer, alors qu'un `.txt` peut être relu, comparé d'une version à l'autre et joint au dossier d'épreuve.

---

## 13. Avancement

- [x] Analyse du contexte et choix de l'architecture réseau
- [x] Liste du matériel Packet Tracer
- [x] Schéma de l'infrastructure
- [x] Plan des VLAN et stratégie HSRP / STP
- [ ] Maquette Packet Tracer : câblage
- [ ] Maquette Packet Tracer : configuration
- [ ] Tests de bascule documentés (captures)
- [ ] Comparatifs des briques systèmes et sécurité
- [ ] Analyse RGPD et NIS2
- [ ] Chiffrage économique (CAPEX et OPEX sur 5 ans)
- [ ] Dossier final et support de présentation
