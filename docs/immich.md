# Immich (Alternative à Google Photos)

Ce document détaille l'installation de la solution de sauvegarde de photos **Immich** et la configuration permettant de déporter le stockage des médias vers le NAS (TrueNAS).

---

## Rôle dans l'infrastructure

Immich est déployé au sein d'un conteneur LXC sur Proxmox. Cette instance utilise une architecture de stockage déporté : les médias sont externalisés sur le NAS via un partage NFS, permettant de séparer la couche applicative (Proxmox) des données (NAS).

---

## Configuration du stockage (Hôte Proxmox)

La première étape consiste à lier le stockage réseau monté sur Proxmox (via NFS) directement au conteneur LXC en utilisant un point de montage (`mp`).

### Commande d'allocation du stockage

Depuis le shell de l'hôte Proxmox :

```bash
# Arrêt du conteneur
pct stop 107

# Liaison du stockage hôte vers le conteneur
# /mnt/pve/immich_storage : Source sur Proxmox (Lien vers TrueNAS)
# /mnt/nas_photos : Destination sur le conteneur
pct set 107 -mp0 /mnt/pve/immich_storage,mp=/mnt/nas_photos

# Redémarrage du conteneur
pct start 107
```

---

## Configuration interne (Conteneur LXC)

Une fois le point de montage actif, le système interne d'Immich doit être redirigé vers ce nouvel espace.

### 1. Création des liens symboliques

Pour assurer la continuité des services d'application et de machine learning sans modifier l'arborescence native d'Immich :

```bash
# Redirection du dossier d'upload vers le NAS
ln -s /mnt/nas_photos/ /opt/immich/app/upload

# Redirection du cache de machine learning vers le NAS
ln -s /mnt/nas_photos/ /opt/immich/app/machine-learning/
```

### 2. Copie des fichiers existants

Les répertoires nécessaires au fonctionnement d'Immich sont copiés vers le nouveau support :

```bash
cp -rv /opt/immich/upload/. /mnt/nas_photos/
```

### 3. Configuration des variables d'environnement

Modification du fichier `.env` pour définir l'emplacement racine des médias :

```bash
nano /opt/immich/.env
# Mise à jour de la variable suivante :
IMMICH_MEDIA_LOCATION=/mnt/nas_photos/
```

---

## Permissions et Droits d'accès

Afin de garantir l'accès aux fichiers pour Immich, les privilèges du système de fichiers sont réattribués à l'utilisateur **immich** au sein du conteneur :

```bash
# Attribution de la propriété au compte de service Immich
chown -R immich:immich /mnt/nas_photos/
```

---

## Avantages de cette architecture

* **Scalabilité** : La capacité de stockage de photos n'est pas limitée par le disque système de Proxmox, mais par les disques installés dans le NAS.
* **Sécurité des données** : En cas de problème sur l'hôte Proxmox, l'intégralité des photos reste intacte sur le NAS.
* **Performance** : L'application (base de données et page web) reste sur Proxmox pour la réactivité, tandis que les fichiers lourds sont déportés sur le réseau.
