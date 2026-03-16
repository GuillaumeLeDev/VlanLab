

 Présentation du Projet

Ce projet a été réalisé dans le cadre de ma préparation au Master of Science Cloud chez EPITECH Lyon. Il simule une architecture réseau d'entreprise multisites, mettant en œuvre la segmentation par VLAN et le routage centralisé.

Objectif : Permettre la communication sécurisée entre deux départements (Comptabilité et Marketing) répartis sur deux commutateurs distincts, tout en isolant les flux au niveau de la couche 2.

 Topologie Réseau


Note : [Insère ici le lien de l'image que tu auras uploadée dans ton dépôt GitHub]

Technologies & Protocoles

Segmentation : VLAN 10 (Comptabilité), VLAN 20 (Marketing), VLAN 99 (Natif).

Transport : Trunking IEEE 802.1Q pour le transport multi-VLAN entre commutateurs.

Routage : Router-on-a-Stick (Sous-interfaces virtuelles sur un lien unique).

Adressage : IPv4 statique avec gestion des passerelles par défaut (Default Gateways).

Extraits de Configuration (Cisco IOS)

1. Configuration du Routeur (Passerelles Inter-VLAN)

Le routeur agit comme le cerveau du réseau, traitant les paquets étiquetés provenant des différents VLANs.

``
interface GigabitEthernet0/0/0.10
 description Gateway VLAN 10 (Comptabilite)
 encapsulation dot1Q 10
 ip address 192.168.10.254 255.255.255.0
!
interface GigabitEthernet0/0/0.20
 description Gateway VLAN 20 (Marketing)
 encapsulation dot1Q 20
 ip address 192.168.20.254 255.255.255.0
``

2. Configuration du Switch (Trunking & Affectation)
``
Le port montant (Uplink) vers le routeur et le lien inter-switch sont configurés en mode Trunk pour laisser passer tous les tags VLAN.

interface GigabitEthernet0/1
 description Uplink vers Routeur
 switchport mode trunk
 switchport trunk native vlan 99
!
interface FastEthernet0/10
 description Acces PC0 (VLAN 10)
 switchport mode access
 switchport access vlan 10


 Diagnostic et Résolution (Troubleshooting)``

Lors du déploiement, j'ai identifié et résolu plusieurs problématiques courantes :

Mismatch de VLAN : Correction de l'affectation des ports physiques pour correspondre au plan d'adressage logique.

Encapsulation : Vérification de l'activation du protocole dot1Q sur les sous-interfaces du routeur.

Connectivité : Validation via des tests de ping et de traceroute entre les différents sous-réseaux.

À propos de moi

Je suis Guillaume Brunel, étudiant en Master Cloud à Epitech Lyon. Je recherche activement une alternance en Ingénierie Réseaux & Systèmes à partir de mars 2026.

LinkedIn : linkedin.com/in/guillaume-dev

Portfolio : github.com/GuillaumeLeDev

Ce lab a été conçu sur Cisco Packet Tracer.
