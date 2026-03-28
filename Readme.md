# 🔀 Lab VLAN & Routage Inter-VLAN (Router-on-a-Stick)

> Simulation d'une architecture réseau d'entreprise multisites avec segmentation VLAN et routage centralisé — réalisé sur Cisco Packet Tracer.

## 🎯 Objectifs du projet

Simuler un réseau d'entreprise où deux départements (Comptabilité et Marketing) doivent :

1. **Être isolés au niveau couche 2** (segmentation par VLAN)
2. **Communiquer entre eux via un routeur** (routage inter-VLAN)
3. **Fonctionner sur deux switches distincts** reliés par un lien trunk

## 🏗️ Architecture réseau

```
                    ┌──────────────┐
                    │   Router0    │
                    │              │
                    │  Gig0/0/0.10 │ ← 192.168.10.254 (GW VLAN 10)
                    │  Gig0/0/0.20 │ ← 192.168.20.254 (GW VLAN 20)
                    └──────┬───────┘
                           │ Gig0/1 (trunk)
                           │
                    ┌──────┴───────┐          Trunk 802.1Q          ┌──────────────┐
                    │   Switch1    │◄────────────Fa0/4──────────────►│   Switch3    │
                    │  2960-24TT   │         VLAN 10,20,99          │    2960      │
                    └──┬───────┬───┘                                └──┬───────┬───┘
                       │       │                                       │       │
                    Fa0/2    Fa0/11                                  Fa0/10   Fa0/20
                       │       │                                       │       │
                 ┌─────┴──┐ ┌──┴──────┐                         ┌─────┴──┐ ┌──┴──────┐
                 │  PC0   │ │Laptop1  │                         │  PC1   │ │Laptop0  │
                 │VLAN 10 │ │VLAN 20  │                         │VLAN 10 │ │VLAN 20  │
                 │ .10.1  │ │ .20.1   │                         │ .10.2  │ │ .20.2   │
                 └────────┘ └─────────┘                         └────────┘ └─────────┘
                  Compta     Marketing                           Compta     Marketing
```

![Topologie Packet Tracer](screenshots/01-topologie-reseau.jpg)
*Topologie complète dans Cisco Packet Tracer — 1 routeur, 2 switches, 4 postes*

## 📋 Plan d'adressage

| Équipement | VLAN | Interface | Adresse IP | Masque | Passerelle |
|------------|------|-----------|------------|--------|------------|
| Router0 | 10 | Gig0/0/0.10 | 192.168.10.254 | 255.255.255.0 | — |
| Router0 | 20 | Gig0/0/0.20 | 192.168.20.254 | 255.255.255.0 | — |
| PC0 | 10 (Compta) | Fa0 | 192.168.10.1 | 255.255.255.0 | 192.168.10.254 |
| PC1 | 10 (Compta) | Fa0 | 192.168.10.2 | 255.255.255.0 | 192.168.10.254 |
| Laptop1 | 20 (Marketing) | Fa0 | 192.168.20.1 | 255.255.255.0 | 192.168.20.254 |
| Laptop0 | 20 (Marketing) | Fa0 | 192.168.20.2 | 255.255.255.0 | 192.168.20.254 |

## 📋 Stack technique

| Composant | Détail |
|-----------|--------|
| Simulateur | Cisco Packet Tracer |
| Routeur | Cisco (Router-on-a-Stick) |
| Switches | Cisco 2960 (x2) |
| VLANs | VLAN 10 (Comptabilité), VLAN 20 (Marketing), VLAN 99 (Natif) |
| Trunking | IEEE 802.1Q |
| Routage | Inter-VLAN via sous-interfaces |
| Adressage | IPv4 statique |

## 🔧 Étapes clés de configuration

### 1. Configuration des VLANs sur les switches

Création des VLANs 10 (Comptabilité), 20 (Marketing) et 99 (Natif) sur les deux switches, puis affectation des ports aux bons VLANs :

```
vlan 10
 name vlan_10
vlan 20
 name vlan_20
vlan 99
 name Native
!
interface FastEthernet0/2
 switchport mode access
 switchport access vlan 10
!
interface FastEthernet0/11
 switchport mode access
 switchport access vlan 20
```

![VLANs Switch 1](screenshots/03-config-switch1-vlans.png)
*Switch 1 — VLAN 10 (Fa0/2, Fa0/10), VLAN 20 (Fa0/11), VLAN 99 (Natif)*

![VLANs Switch 2](screenshots/04-config-switch2-vlans.png)
*Switch 3 — VLAN 10 (Fa0/10), VLAN 20 (Fa0/11, Fa0/20)*

### 2. Configuration du Trunking 802.1Q

Les liens entre switches et entre switch-routeur sont configurés en trunk pour transporter les trames étiquetées de tous les VLANs :

```
interface FastEthernet0/4
 description Lien inter-switch (trunk)
 switchport mode trunk
 switchport trunk native vlan 99
!
interface GigabitEthernet0/1
 description Uplink vers Routeur
 switchport mode trunk
 switchport trunk native vlan 99
```

![Status Trunk](screenshots/05-trunk-status.png)
*Ports trunk actifs — Fa0/4 (inter-switch) et Gig0/1 (vers routeur), encapsulation 802.1Q, VLAN natif 99*

### 3. Routage Inter-VLAN (Router-on-a-Stick)

Le routeur utilise des sous-interfaces sur un seul lien physique pour router le trafic entre les VLANs. Chaque sous-interface est la passerelle par défaut de son VLAN :

```
interface GigabitEthernet0/0/0.10
 description Gateway VLAN 10 (Comptabilite)
 encapsulation dot1Q 10
 ip address 192.168.10.254 255.255.255.0
!
interface GigabitEthernet0/0/0.20
 description Gateway VLAN 20 (Marketing)
 encapsulation dot1Q 20
 ip address 192.168.20.254 255.255.255.0
```

![Config Routeur](screenshots/02-config-routeur.png)
*Sous-interfaces du routeur — .10 (192.168.10.254) et .20 (192.168.20.254) UP*

## ✅ Tests de validation

### Ping inter-VLAN (Comptabilité → Marketing)

Depuis un PC du VLAN 10 (Comptabilité), ping vers un PC du VLAN 20 (Marketing) — le trafic doit passer par le routeur :

![Ping inter-VLAN](screenshots/06-ping-inter-vlan.png)
*Ping de 192.168.10.x vers 192.168.20.2 — succès, le routage inter-VLAN fonctionne*

### Ping inter-switch (même VLAN, switches différents)

Depuis PC0 (192.168.10.1, Switch 1) vers PC1 (192.168.10.2, Switch 3) — vérifie que le trunk transporte bien les trames VLAN entre switches :

![Ping inter-switch](screenshots/07-ping-inter-switch.png)
*Ping 192.168.10.1 → 192.168.10.2 — 0% perte, TTL=128, le trunk fonctionne*

### Traceroute inter-VLAN

Le traceroute prouve que le trafic entre VLANs passe bien par le routeur (Router-on-a-Stick) :

![Traceroute](screenshots/08-traceroute.png)
*Traceroute VLAN 10 → VLAN 20 : hop 1 = 192.168.10.254 (routeur), hop 2 = 192.168.20.2 (destination)*

> **Note** : le hop 1 (`192.168.10.254`) est la sous-interface du routeur côté VLAN 10. C'est la preuve que le trafic inter-VLAN transite par le routeur — il n'y a pas de communication directe entre VLANs au niveau des switches (isolation couche 2 respectée).

## 🚧 Difficultés rencontrées et solutions

### Mismatch de VLAN
**Problème** : certains PCs ne communiquaient pas car leurs ports physiques n'étaient pas assignés au bon VLAN.
**Diagnostic** : vérification avec `show vlan brief` pour identifier les ports mal affectés.
**Solution** : correction de l'affectation des ports avec `switchport access vlan [ID]`.

### Encapsulation dot1Q
**Problème** : le routage inter-VLAN ne fonctionnait pas initialement.
**Diagnostic** : les sous-interfaces du routeur n'avaient pas l'encapsulation dot1Q activée.
**Solution** : ajout de `encapsulation dot1Q [VLAN_ID]` sur chaque sous-interface avant l'assignation d'IP.

### Validation de connectivité
**Méthode** : tests systématiques par ping et traceroute entre les différents sous-réseaux pour valider chaque étape de la configuration.

## 📚 Ce que j'ai appris

- **Segmentation VLAN** : isoler les flux réseau au niveau couche 2 pour sécuriser et organiser le trafic
- **Trunking 802.1Q** : transporter plusieurs VLANs sur un seul lien physique grâce à l'étiquetage des trames
- **Router-on-a-Stick** : utiliser des sous-interfaces virtuelles sur un routeur pour assurer le routage inter-VLAN sans multiplier les interfaces physiques
- **VLAN natif** : comprendre le rôle du VLAN 99 comme VLAN natif pour le trafic non étiqueté sur les trunks
- **Diagnostic Cisco IOS** : utiliser `show vlan brief`, `show interfaces trunk`, `show ip interface brief` pour identifier et résoudre les problèmes

## 🛠️ Compétences démontrées

`VLAN` · `Trunking 802.1Q` · `Router-on-a-Stick` · `Routage Inter-VLAN` · `Cisco IOS` · `Cisco Packet Tracer` · `Segmentation réseau` · `Adressage IPv4` · `Diagnostic réseau` · `Configuration switches & routeurs`

## 📌 Évolutions possibles

- [ ] Ajouter un troisième département (VLAN 30 — RH) avec des règles ACL pour restreindre l'accès
- [ ] Mettre en place du DHCP sur le routeur pour l'attribution automatique des adresses
- [ ] Configurer du Port Security sur les switches pour limiter les adresses MAC autorisées
- [ ] Ajouter un serveur dans une DMZ avec des ACL spécifiques
- [ ] Implémenter STP (Spanning Tree Protocol) avec un switch racine défini

---

*Projet réalisé dans le cadre de la préparation au Master of Science Cloud à EPITECH Lyon — en recherche d'alternance en administration réseau / sécurité IT.*
