# /!\ UNDER CONSTRUCTION : README.md OUT-OF-DATE

# OT Cyber Range

Cyber-range OT modulaire basé sur QEMU/KVM, Ansible, Docker, nftables et ufw.

L’objectif du projet est de fournir un laboratoire reproductible permettant de :
- déployer une segmentation réseau inspirée du modèle Purdue,
- créer automatiquement des VMs sous QEMU/KVM,
- déployer les services via Docker,
- centraliser la configuration dans Git,
- faciliter l’ajout ou le remplacement de briques OT/IT.

## Arborescence

```
ot-lab/
├── deploy.yml
├── inventory/
│   └── lab.yml
├── roles/
│   ├── network/
│   │   ├── tasks/main.yml
│   │   └── templates/
│   │       └── networks.xml.j2
│   └── vm/
│       ├── tasks/main.yml
│       └── templates/
│           └── vm.xml.j2
├── vars/
│   ├── networks.yml
│   └── vms.yml
├── images/
│   ├── debian-genericcloud.qcow2
│   └── debian-genericcloud.sha256
├── iso/
│   └── windows-server-2022.iso
└── README.md

```
---
## Prérequis

### Hôte de déploiement
- Linux
- QEMU/KVM
- libvirt
- Ansible
- Python 3
- Docker / Docker Compose

### Paquets recommandés
- qemu-kvm
- libvirt-daemon-system
- libvirt-clients
- virtinst
- ansible
- python3-lxml
- docker
- docker compose

### Collections Ansible recommandées
- community.libvirt

### Médias nécessaires
- images/debian-genericcloud.qcow2
- images/debian-genericcloud.sha256
- iso/windows-server-2022.iso

---
## Installation

1. Cloner le dépôt
git clone git@github.com:TheoCaudan/OT-cyberrange.git
cd OT-cyberrange

2. Vérifier les médias
Placer les fichiers dans :
- images/
- iso/

Puis vérifier le checksum si nécessaire.

3. Vérifier la configuration
Adapter si besoin :
- vars/networks.yml
- vars/vms.yml
- inventory/lab.yml

4. Lancer le déploiement
ansible-playbook -i inventory/lab.yml deploy.yml -K

## Ajout ou modification de configuration

### Ajouter une VM
1. Ouvrir vars/vms.yml
2. Ajouter une nouvelle entrée dans la liste vms
3. Définir :
   - name
   - cpu
   - ram_mb
   - disk_gb
   - type
   - base_image
   - networks

### Ajouter un réseau
1. Ouvrir vars/networks.yml
2. Ajouter une nouvelle entrée dans la liste networks
3. Définir :
   - name
   - bridge
   - gateway
   - netmask
   - forward_mode

### Modifier un pare-feu
Les règles de filtrage doivent être définies dans le rôle correspondant et/ou dans les variables de la VM concernée.

Recommandation :
- garder nftables comme politique de filtrage principale
- utiliser ufw pour des ajustements simples d’exploitation

Ajouter un service Docker à une VM
1. Préparer le dossier Docker du service
2. Ajouter le rôle ou la tâche Ansible correspondante
3. Mettre à jour la configuration de la VM si nécessaire
4. Relancer le playbook

---
## Philosophie du projet
- Git = source de vérité
- Ansible = orchestration
- libvirt/QEMU = virtualisation
- Docker = isolation applicative
- nftables = politique de filtrage
- ufw = administration simplifiée

## Notes
Ce projet est conçu pour être modulaire.
Les composants peuvent être remplacés progressivement selon les besoins du labo, par exemple :
- protocole OT
- services SCADA
- outils de supervision
- scénarios de détection
