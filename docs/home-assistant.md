# Home Assistant

Ce document détaille l'implémentation de Home Assistant au sein du HomeLab et son interaction avec le réseau local via le protocole MQTT.

---

## Rôle dans l'infrastructure

Home Assistant (HA) agit comme le superviseur central de la domotique. Il est configuré pour être le point de convergence et de traitement des données provenant des capteurs Zigbee, tout en restant indépendant de la couche matérielle de réception.

1. **Centralisation** : Regroupement de tous les appareils connectés sur une interface unique pour une gestion unifiée.
2. **Automatisation** : Exécution de scripts et de scénarios basés sur les événements réseau ou les changements d'état des capteurs.
3. **Souscription MQTT** : Récupération et interprétation des messages publiés sur le broker par le service Zigbee2MQTT.

---

## Déploiement sur Proxmox

Home Assistant est installé dans une Machine Virtuelle (VM) dédiée pour garantir une stabilité maximale et l'accès aux fonctionnalités de Home Assistant OS (HAOS).

### Ressources allouées
* **CPU** : 2 vCPUs
* **RAM** : 4 Go
* **Stockage** : 32 Go 



---

## Flux de communication (MQTT)

L'intégration des périphériques repose sur une architecture découplée utilisant le broker Mosquitto.

1. **Acquisition** : Le dongle Zigbee est géré par un conteneur LXC indépendant (Zigbee2MQTT).
2. **Transport** : Les messages sont transmis au broker Mosquitto (LXC) sous forme de topics.
3. **Traitement** : Home Assistant est configuré comme client MQTT pour s'abonner aux topics et mettre à jour l'état des entités en temps réel.

---

## Gestion des données
* **Base de données** : Utilisation de la base SQLite interne à Home Assisant pour la gestion de l'historique et des états.
* **Évolutivité** : L'envoie des sauvegardes vers le Lenovo ThinkCentre (TrueNAS) est prévu pour une meilleure persistance des données.





