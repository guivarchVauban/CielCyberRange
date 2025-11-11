# Installation de Debian avec FreePBX sur Proxmox

Ce guide explique comment créer et configurer une machine virtuelle Debian avec FreePBX pour la téléphonie IP sur **Proxmox VE**.  

**Caractéristiques de la VM :**
- OS : Debian (dernière version stable)
- FreePBX (interface Asterisk)
- 2 vCPU
- 3 Go de RAM
- 20 Go de disque
- Interfaces réseau :
  - VLAN 30 → VoIP
  - VLAN 10 → Administration

---

## 1. Création de la VM sur Proxmox

1. Connectez-vous à l’interface Proxmox VE.
2. Cliquez sur **Créer VM**.
3. Paramètres de base :
   - **Node** : votre serveur Proxmox
   - **VM ID** : auto-généré ou choisi
   - **Nom** : Debian-FreePBX
4. **OS** :
   - ISO : sélectionner l’image Debian stable (ex. `debian-12.3.0-amd64-netinst.iso`)
   - Type : Linux
5. **System** :
   - BIOS : Default (SeaBIOS)
   - Machine : Default (q35)
6. **Disque** :
   - Taille : 20 Go
   - Format : qcow2
   - Storage : sélectionner le stockage souhaité
7. **CPU** :
   - Sockets : 1
   - Cores : 2
8. **Mémoire** :
   - RAM : 3 Go
9. **Réseau** :
   - Mode : VirtIO
   - VLAN Tag : 10 pour l’interface admin, 30 pour VoIP (ajouter une seconde interface réseau pour VoIP)
10. Valider et créer la VM.

---

## 2. Installation de Debian

1. Démarrer la VM avec l’ISO Debian.
2. Choisir **Install** ou **Graphical Install**.
3. Configurer la langue, le fuseau horaire et le clavier.
4. Réseau :
   - Interface admin → DHCP ou IP fixe sur VLAN 10
   - Interface VoIP → IP fixe sur VLAN 30
5. Nom de la machine et domaine : par exemple `freepbx.local`
6. Utilisateur et mot de passe :
   - Root : définir un mot de passe sécurisé
7. Partitionnement :
   - Utiliser **Guidé – utiliser tout le disque**
8. Installer le système de base et SSH server. **Décocher toutes les interfaces de bureau**

---

## 3. Préparation pour FreePBX

1. Mettre à jour Debian :
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```
2. Installer les paquets nécessaires :
   ```bash
   sudo apt install -y wget curl gnupg2 lsb-release sudo
   ```

---

## 4. Installation de FreePBX / Asterisk

1. Télécharger le script d’installation de FreePBX :
   ```bash
   cd /usr/src
   sudo wget http://mirror.freepbx.org/modules/packages/freepbx/freepbx-16.0-latest.tgz
   sudo tar xfz freepbx-16.0-latest.tgz
   cd freepbx
   ```
2. Installer les dépendances :
   ```bash
   sudo ./start_asterisk start
   sudo ./install -n
   ```
3. Configurer FreePBX via le navigateur :
   - URL : `http://IP_DE_LA_VM`
   - Créer l’utilisateur admin pour l’interface web.

---

## 5. Configuration réseau et sécurité

- Vérifier que les deux interfaces réseau sont configurées correctement :
  ```bash
  ip a
  ```
- Activer le firewall pour restreindre l’accès au port admin (80/443) et aux ports VoIP (SIP/RTP).
- Configurer NAT et QoS si nécessaire pour la VoIP.

---

## 6. Tests et validation

1. Accéder à l’interface FreePBX :
   ```
   http://IP_ADMIN
   ```
2. Créer une extension SIP et tester un appel interne.
3. Vérifier que les VLAN fonctionnent :
   - Ping depuis VLAN admin → VLAN VoIP
   - Test SIP vers l’extérieur si nécessaire

---

### Notes

- FreePBX recommande Debian 12 stable pour la dernière version 16.
- Les VLAN doivent être configurés côté Proxmox et switch managé.
- Toujours sauvegarder la VM après installation et configuration initiale.

