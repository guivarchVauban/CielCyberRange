# TP Cyberange - Architecture Proxmox + OPNsense + Docker + FreePBX

## Compte-rendu
Toutes les étapes de configuration (adresses IP attribuées, VLAN et ports du switch, identifiants et mots de passe créés, règles firewall, paramètres Docker, etc.) doivent être consignées précisément dans un fichier COMPTE_RENDU.md et déposées sur le serveur GitLab de la section. Ce compte rendu doit être à jour, complet et permettre à un autre groupe de reproduire intégralement votre installation.

## Objectif
Mettre en place une architecture réseau réaliste comprenant :
- Un routeur/firewall **OPNsense** avec gestion des VLANs
- Un serveur **FreePBX** pour la téléphonie VoIP
- Un **Docker Host** hébergeant GLPI, Wazuh et Portainer
- Le tout virtualisé sous **Proxmox**

---

## VLANs utilisés

| VLAN | Rôle | ID | Exemple Subnet |
|------|------|----|----------------|
| WAN / Administration (trunk vers prof) | Accès admin / sortie pédagogique | **1** | 10.0.1.0/24 |
| Administration (gestion serveurs) | Accès consoles Proxmox, OPNsense, Portainer | 10 | 10.10.10.0/24 |
| Clients | Postes utilisateurs | 20 | 10.20.20.0/24 |
| VoIP | Téléphones / FreePBX | 30 | 10.30.30.0/24 |
| DMZ | Services exposés (web) | 40 | 10.40.40.0/24 |

---

## Répartition des machines (VMs Proxmox) et allocation mémoire (sur hôte 16 Go RAM)

**Remarque importante** : Réserver environ **2–3 Go pour Proxmox lui-même** (gestion, cache).

| VM | Rôle | vCPU | RAM | Disque | Notes |
|----|------|------|-----|--------|------|
| **OPNsense** | Firewall + Routage inter-VLANs | 2 vCPU | **2 Go** | 50 Go | Interfaces virtuelles vtnet0..vtnet4 |
| **Debian + FreePBX** | Téléphonie IP (Asterisk/FreePBX) | 2 vCPU | **3 Go** | 20 Go | Interface sur VLAN 30 (VoIP) |
| **Debian Docker Host** | GLPI + Wazuh + Portainer | 4 vCPU | **8 Go** | 100 Go | Interface sur VLAN 10 (Administration) et VLAN 20 (Clients) |
| **Poste Admin Kali (optionnel)** | Admin / Prof | 1 vCPU | **1 Go** | 10 Go | Interface sur VLAN 1 (WAN/Admin) ou VLAN 10 |

**Total RAM allouée : ≈ 14 Go** (laisse 2 Go libres pour la machine hôte).

---

## Mapping des interfaces OPNsense (conforme à ta demande)
OPNsense disposera de **5 interfaces virtuelles** (vtnet0 → vtnet4), chacune mappée à un VLAN précis sur `vmbr0` (bridge VLAN-aware) :

| Interface OPNsense | VM Interface (Proxmox) | VLAN Tag sur vmbr0 | Rôle |
|--------------------|-------------------------|--------------------:|------|
| `vtnet0` | net0 | **1** | WAN (accès depuis réseau pédagogique / accès Proxmox) |
| `vtnet1` | net1 | **10** | Administration (gestion serveurs) |
| `vtnet2` | net2 | **20** | Clients |
| `vtnet3` | net3 | **30** | VoIP (FreePBX) |
| `vtnet4` | net4 | **40** | DMZ (Web public) |

> Dans Proxmox, ajoute 5 *Network Device* pour la VM OPNsense : Bridge = `vmbr0`, et **VLAN Tag** = 1, 10, 20, 30, 40 respectivement.
> À l'intérieur d'OPNsense, ces interfaces apparaîtront comme `vtnet0` à `vtnet4`. Lors du premier démarrage, affecte `vtnet0` comme WAN, puis assigne et active les autres interfaces via **Interfaces → Assignments**.

---

## Configuration recommandée dans Proxmox (interface Web)
1. **Créer `vmbr0`** :
   - Datacenter → Node → Network → Create → Linux Bridge
   - Name: `vmbr0`
   - Bridge ports: `enp0s3` (ou ton interface physique)
   - Coche **VLAN aware**
   - Create

2. **Ajouter les interfaces à la VM OPNsense** :
   - VM OPNsense → Hardware → Add → Network Device
   - Bridge: `vmbr0`
   - VLAN Tag: `1` → Add
   - Répéter avec VLAN Tag `10`, `20`, `30`, `40` pour les autres netX

3. **Ajouter interfaces aux autres VMs** :
   - Docker Host: Bridge `vmbr0`, VLAN Tag `10`
   - FreePBX VM: Bridge `vmbr0`, VLAN Tag `30`
   - Poste Admin: Bridge `vmbr0`, VLAN Tag `1` (si tu veux accéder à Proxmox via VLAN1)

4. **Apply Configuration** si demandé.

## Règles réseau essentielles à créer dans OPNsense
- **Par défaut** : DENY all entre VLANs.
- Autoriser NAT sortant pour VLANs vers WAN (vtnet0)
- Pour le site web en DMZ : configurer NAT/port forwarding depuis WAN→DMZ (ports 80/443)
---

## Liens utiles :
- [Installer et configurer OPNsense sur Proxmox](https://github.com/guivarchVauban/CielCyberRange/blob/main/Installation%20et%20configuration%20de%20OPNsense.md)
- [Installer Debian Serveur + Docker + Portainer](https://github.com/guivarchVauban/CielCyberRange/blob/main/Installation%20Debian%20Server%20%2B%20Docker%20%2B%20Portainer.md)
- [Installer Wazuh avec Docker](https://documentation.wazuh.com/current/deployment-options/docker/wazuh-container.html) 
   - Attention suivre la procédure **Single node stack deployement**
- [Installer GLPI avec Docker](https://github.com/guivarchVauban/glpi_docker)
