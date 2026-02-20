# HomeLab : Infrastructure Domotique & Stockage

[![Proxmox](https://img.shields.io/badge/Hypervisor-Proxmox_VE-orange?logo=proxmox)](https://www.proxmox.com)
[![Tailscale](https://img.shields.io/badge/VPN-Tailscale-blue?logo=tailscale)](https://tailscale.com/)
[![TrueNAS](https://img.shields.io/badge/Storage-TrueNAS_Scale-0095D5?logo=truenas)](https://www.truenas.com/)
[![HomeAssistant](https://img.shields.io/badge/SmartHome-Home_Assistant-blue?logo=home-assistant)](https://www.home-assistant.io/)

Ce dépôt documente mon infrastructure hybride combinant virtualisation, domotique et stockage réseau (NAS). Ce projet me permet d'automatiser ma maison tout en maintenant un environnement de test pour l'administration système.

---

## Architecture Matérielle (Hardware)

L'infrastructure repose sur deux nœuds physiques distincts pour séparer le calcul et le stockage :

| Équipement | Modèle | OS | Usage |
| :--- | :--- | :--- | :--- |
| **Node 1** | **Intel NUC** | **Proxmox VE** | Hyperviseur (VM & LXC), Domotique |
| **Node 2** | **Lenovo ThinkCentre** | **TrueNAS SCALE** | Stockage centralisé (ZFS), Backups, SMB/NFS |

---

## Services & Virtualisation (Intel NUC)

### Réseau & Accès Distant (Tailscale)
Pour l'accès distant sécurisé sans ouverture de ports (NAT traversal), j'utilise un **LXC Tailscale**.
* **Subnet Router :** Permet d'accéder à l'ensemble du LAN `192.168.1.0/24` à distance.

### Écosystème Domotique
* **Home Assistant (VM) :** Système central gérant l'automatisation.
* **Zigbee2MQTT (LXC) :** Passerelle pour les équipements Zigbee (Dongle USB Sonoff).
* **Mosquitto (LXC) :** Broker MQTT servant de bus de communication.


## Supervision et Gestion

* **InfluxDB (LXC)** : Base de données temporelle collectant les métriques de performance à long terme.
* **Grafana (LXC)** : Visualisation de l'historique (températures, charge CPU).
* **Proxmenux** : Interface web de monitoring offrant une vue d'ensemble simplifiée et en temps réel des statistiques de l'hôte Proxmox.

---

## Stockage et Partages (Lenovo ThinkCentre)

Le Lenovo ThinkCentre, sous TrueNAS, assure la gestion centralisée des données du HomeLab :

1. **ZFS Pool** : Configuration en disques isolés (Single-disk VDEVs) pour limiter l'impact d'une panne matérielle à un seul volume.
2. **SMB Shares** : Partages de fichiers personnels configurés pour un accès multiplateforme (Windows, Linux).
3. **Connectivité** : Accès distant sécurisé via le Subnet Router Tailscale hébergé sur le Intel NUC.

## Sauvegarde et Résilience

L'infrastructure est conçue pour séparer le calcul du stockage des données critiques :

* **Externalisation** : Les sauvegardes de l'ensemble des conteneurs LXC et machines virtuelles du Intel NUC sont exportées vers le TrueNas (Lenovo ThinkCentre).
* **Protocole** : Utilisation d'un point de montage NFS entre Proxmox et TrueNAS pour assurer des transferts rapides et une gestion native par l'hyperviseur.
* **Résilience** : Cette architecture permet une restauration rapide des services sur un noeud tiers en cas de défaillance matérielle du NUC.