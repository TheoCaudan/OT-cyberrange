# OT Cyber Range
---
## Cyber-range OT modulaire basé sur QEMU/KVM, Ansible, Docker, nftables et UFW.

L’objectif du projet est de fournir un laboratoire reproductible permettant de :
- déployer une segmentation réseau inspirée du modèle Purdue,
- créer automatiquement des VMs sous QEMU/KVM,
- déployer des services applicatifs via Docker,
- centraliser la configuration dans Git,
- faciliter l’ajout, le remplacement ou l’évolution des briques IT/OT.

### Vue d’ensemble

Le lab est découpé en plusieurs zones réseau :
- IT
- DMZ
- OTL3
- OTL2

La logique générale suit une approche Purdue :
- IT ne parle pas directement aux couches OT,
- DMZ sert de zone tampon,
- bastion comme point d’accès contrôlé,
- OTL3 et OTL2 ne reçoivent que les flux nécessaires,
- Modbus TCP est utilisé entre OTL3 et OTL2.

### Fonctionnalités principales
- Déploiement automatisé des réseaux libvirt
- Déploiement automatisé des VMs
- Support de VMs Linux cloud-init
- Support de VMs Windows à partir d’un ISO
- Déploiement de services Docker :
  - Bastion / Guacamole
  - SIEM
  - SCADA / Ignition
  - PLC / OpenPLC + worker Python
- Séparation claire des variables :
  - topologie réseau
  - ressources VM
  - configuration applicative par rôle
  - secrets via Vault
- Architecture modulaire pour faire évoluer le lab par morceaux

### Arborescence

ot-lab/
├── deploy.yml
├── inventory/
│   └── lab.yml
├── roles/
│   ├── network/
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   └── templates/
│   │       └── networks.xml.j2
│   ├── vm/
│   │   ├── tasks/
│   │   │   ├── main.yml
│   │   │   ├── linux.yml
│   │   │   └── windows.yml
│   │   └── templates/
│   │       ├── vm.xml.j2
│   │       ├── user-data.j2
│   │       └── meta-data.j2
│   ├── firewall/
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   ├── handlers/
│   │   │   └── main.yml
│   │   └── templates/
│   │       ├── nftables.conf.j2
│   │       └── ufw.conf.j2
│   ├── bastion/
│   │   ├── defaults/
│   │   │   └── main.yml
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   ├── handlers/
│   │   │   └── main.yml
│   │   └── templates/
│   │       ├── docker-compose.yml.j2
│   │       └── bastion.env.j2
│   ├── siem/
│   │   ├── defaults/
│   │   │   └── main.yml
│   │   ├── tasks/
│   │   │   ├── main.yml
│   │   │   ├── install.yml
│   │   │   └── deploy.yml
│   │   ├── handlers/
│   │   │   └── main.yml
│   │   └── templates/
│   │       ├── wazuh-docker-compose.yml.j2
│   │       ├── suricata-docker-compose.yml.j2
│   │       ├── ossec.conf.j2
│   │       └── suricata.yaml.j2
│   ├── ad-dc/
│   │   ├── tasks/
│   │   │   ├── main.yml
│   │   │   ├── install.yml
│   │   │   └── populate.yml
│   │   └── templates/
│   │       └── populate_ad.ps1.j2
│   ├── ad-client/
│   │   ├── tasks/
│   │   │   ├── main.yml
│   │   │   ├── install.yml
│   │   │   ├── join_domain.yml
│   │   │   └── post_join.yml
│   │   └── defaults/
│   │       └── main.yml
│   ├── scada/
│   │   ├── defaults/
│   │   │   └── main.yml
│   │   ├── tasks/
│   │   │   ├── main.yml
│   │   │   ├── install.yml
│   │   │   └── deploy.yml
│   │   ├── handlers/
│   │   │   └── main.yml
│   │   └── templates/
│   │       ├── docker-compose.yml.j2
│   │       └── scada.env.j2
│   └── plc/
│       ├── defaults/
│       │   └── main.yml
│       ├── tasks/
│       │   ├── main.yml
│       │   ├── install.yml
│       │   └── deploy.yml
│       ├── handlers/
│       │   └── main.yml
│       └── templates/
│           ├── docker-compose.yml.j2
│           └── plc.env.j2
├── vars/
│   ├── networks.yml
│   ├── vms.yml
│   ├── firewall-it-dmz.yml
│   ├── firewall-dmz-otl3.yml
│   ├── firewall-otl3-otl2.yml
│   ├── siem.yml
│   ├── ad-dc.yml
│   ├── ad-client.yml
│   ├── bastion.yml
│   ├── scada.yml
│   ├── plc.yml
│   └── services.yml
├── host_vars/
│   └── bastion/
│       └── vault.yml
├── images/
│   ├── debian-genericcloud.qcow2
│   └── debian-genericcloud.sha256
├── iso/
│   └── windows-server-2022.iso
└── README.md

---
### Prérequis

#### Hôte de déploiement
- Linux
- QEMU/KVM
- libvirt
- Ansible
- Python 3
- Docker / Docker Compose

#### Paquets recommandés
- qemu-kvm
- libvirt-daemon-system
- libvirt-clients
- virtinst
- ansible
- python3-lxml
- docker
- docker compose

#### Collections Ansible recommandées
- community.libvirt
- ansible.windows si tu continues à automatiser côté Windows

#### Médias nécessaires
- images/debian-genericcloud.qcow2
- images/debian-genericcloud.sha256
- iso/windows-server-2022.iso

---
### Architecture lab

#### Réseaux
Les réseaux attendus sont définis dans vars/networks.yml :
- it
- dmz
- otl3
- otl2

#### Machines principales
- fw-it-dmz
- fw-dmz-otl3
- fw-otl3-otl2
- siem-ids
- dc
- client-ad
- bastion
- scada
- plc

#### Rôles applicatifs
- firewall
- siem
- ad-dc
- ad-client
- bastion
- scada
- plc

---
### Installation

#### 1. Cloner le dépôt
```
git clone git@github.com:TheoCaudan/OT-cyberrange.git
cd OT-cyberrange
```
#### 2. Vérifier les médias
Placer les fichiers dans :
- images/
- iso/

Puis vérifier le checksum si nécessaire.

#### 3. Vérifier la configuration
Adapter si besoin :
- vars/networks.yml
- vars/vms.yml
- vars/*.yml
- inventory/lab.yml

#### 4. Déployer l’infrastructure
```
ansible-playbook -i inventory/lab.yml deploy.yml -K
```
---

### Gestion des variables et des secrets

Le projet suit une séparation simple :
- vars/*.yml : configuration de lab
- host_vars/<host>/vault.yml : secrets, idéalement chiffrés
- defaults/main.yml : valeurs par défaut des rôles

### Exemples de secrets
- mots de passe Guacamole
- identifiants d’administration
- secrets applicatifs éventuels

### Ajout ou modification de configuration

#### Ajouter une VM
1. Ouvrir vars/vms.yml
2. Ajouter une entrée dans la liste vms
3. Définir au minimum :
   - name
   - role
   - zone
   - cpu
   - ram_mb
   - disk_gb
   - type
   - os_family
   - base_image
   - role_vars
   - networks

#### Ajouter un réseau
1. Ouvrir vars/networks.yml
2. Ajouter une entrée dans la liste networks
3. Définir :
   - name
   - bridge
   - subnet
   - gateway
   - netmask
   - dhcp
   - forward_mode
   - autostart

#### Ajouter une configuration métier
1. Créer un fichier dans vars/
2. L’associer à une VM via role_vars
3. Adapter le rôle correspondant si nécessaire

#### Modifier un pare-feu
Les règles sont définies par firewall, dans les fichiers suivants :
- vars/firewall-it-dmz.yml
- vars/firewall-dmz-otl3.yml
- vars/firewall-otl3-otl2.yml

#### Principe général :
- pas de lien direct IT → OT
- DMZ comme zone tampon
- bastion comme point d’accès
- OT limité aux flux nécessaires
- Modbus TCP entre OTL3 et OTL2

#### Ajouter un service Docker à une VM
1. Préparer le rôle correspondant
2. Ajouter un fichier de vars dédié
3. Mettre à jour vms.yml
4. Relancer le playbook

---
### Philosophie du projet

- Git = source de vérité
- Ansible = orchestration
- libvirt/QEMU = virtualisation
- Docker = isolation applicative
- nftables = politique de filtrage principale
- UFW = interface d’exploitation / simplification
- Vault = secrets
- cloud-init = provisioning des Debian cloud images
- ISO = installation Windows

---
### Notes d’architecture

#### Modèle Purdue
Le lab suit une segmentation inspirée du Purdue model :
- IT
- DMZ
- OTL3
- OTL2

#### Accès
- le bastion est l’unique point d’entrée logique vers la partie OT
- IT ne doit pas accéder directement à OTL3/OTL2
- OTL3 ↔ OTL2 est limité au flux industriel nécessaire

OT
- SCADA : Ignition
- PLC : OpenPLC + worker Python maison
- SIEM : Wazuh / Suricata
- AD : contrôleur de domaine + client domaine

---
### Évolution prévue

Le projet est conçu pour être modulaire. Les composants peuvent être remplacés progressivement selon les besoins du labo, par exemple :
- protocole OT
- runtime PLC
- stack SCADA
- règles IDS/IPS
- scénarios de détection
- nouveaux services de lab

#### Statut

Projet encore en cours de stabilisation, mais la structure est conçue pour être :
- reproductible
- modulaire
- lisible
- testable
- évolutive
