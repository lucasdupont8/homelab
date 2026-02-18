
# Tailscale et accès distant (Subnet Router)

Ce document détaille l'implémentation de Tailscale au sein de mon infrastructure HomeLab. Dans cette configuration, Tailscale est utilisé comme un point d'entrée unique et sécurisé, permettant l'administration à distance de l'ensemble des nœuds sans exposition directe sur internet.

---

## Rôle dans l'infrastructure

L'architecture repose sur l'utilisation de Tailscale en tant que **Subnet Router**. Cette approche permet d'accéder à des ressources sur lesquelles l'agent Tailscale n'est pas (ou ne peut pas être) installé.

1. **Passerelle d'administration** : Le conteneur Tailscale sur le Intel NUC sert de pivot pour atteindre l'interface de gestion de Proxmox.
2. **Accès au stockage (TrueNAS)** : Bien que le Lenovo ThinkCentre ne possède pas l'agent Tailscale, il est accessible via son adresse IP locale (192.168.1.X) grâce au routage configuré sur le NUC.
3. **Sécurité réseau** : Évite l'ouverture de ports sur le routeur domestique (NAT Traversal) et chiffre les flux d'administration de bout en bout avec le protocole WireGuard.



---

## Déploiement sur Proxmox (LXC)

Le service est hébergé dans un conteneur LXC sous Debian 12. Pour un fonctionnement optimal en environnement virtualisé, une configuration spécifique de l'hôte est requise.

### 1. Configuration de l'hôte Proxmox

Pour permettre au conteneur de créer une interface réseau virtuelle (tun), le fichier de configuration du conteneur sur l'hôte Proxmox (`/etc/pve/lxc/ID_LXC.conf`) doit inclure les paramètres suivants :

```
lxc.cgroup2.devices.allow: c 10:200 rwm
lxc.mount.entry: /dev/net/tun dev/net/tun none bind,create=file
```

### 2. Installation du client

L'installation est effectuée via le dépôt officiel Tailscale :

```bash
curl -fsSL [https://tailscale.com/install.sh](https://tailscale.com/install.sh) | sh

```

---

## Configuration du routage (Subnet Routing)

Le but est de permettre aux paquets provenant du réseau virtuel Tailscale de transiter vers le réseau physique local (LAN).

### Activation de l'IP Forwarding

Cette modification au niveau du noyau Linux autorise le transfert de paquets entre les interfaces réseau :

```bash
echo 'net.ipv4.ip_forward = 1' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv6.conf.all.forwarding = 1' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

```

### Initialisation du service

La commande suivante enregistre le nœud et annonce les routes vers le sous-réseau local :

```bash
tailscale up --advertise-routes=192.168.1.0/24

```

*Note : Une validation manuelle est nécessaire dans la console d'administration Tailscale ("Edit route settings") pour autoriser le routage du sous-réseau.*

---

## Commandes de maintenance

| Commande | Action |
| --- | --- |
| tailscale status | Liste les machines connectées et leur état. |
| tailscale ip -4 | Affiche l'adresse IP interne du nœud sur le Tailnet. |
| tailscale down | Déconnecte la machine du réseau. |

