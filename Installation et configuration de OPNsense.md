# Installation & Configuration de OPNsense

## 🎯 Objectif
Installer OPNsense comme routeur/firewall pour gérer plusieurs VLANs et fournir l’accès Internet.

| Interface VM | Rôle | VLAN | Adresse |
|---|---|---|---|
| vtnet0 | WAN | 1 | 
| vtnet1 | DMZ | 10 |
| vtnet2 | Clients | 20 |
| vtnet3 | Administration | 30 |
| vtnet4 | VOIP | 40 |


---

## 1) Création de la VM OPNsense dans Proxmox

- **Télécharger le fichier iso.bz2 (choisir le format DVD) depuis le [site d'OPNsense](https://opnsense.org/download/)**

- Extraire le bz2 pour obtenir un .iso et la placer dans le dossier `/var/lib/vz/template/iso`

- Créer une nouvelle VM avec la configuration suivante : 

| Paramètre | Valeur |
|---|---|
| OS | Other |
| CPU | 2 |
| RAM | 2 Go |
| Disque | 50 Go |
| NICs | 5 interfaces sur vmbr0 avec tags VLAN : 1,10,20,30,40 | 

**Les Interfaces seront à ajouter à la main après la création de la VM**

Démarrer la VM et ouvrir une console noVNC
Attendre le démarrage et le prompt de login (ça peut être long)

Taper:(attention le clavier et en Qwerty)
| Login | Mot de passe |
| ---|---|
|installer | opnsense |

Suivre ensuite ce [tutoriel](https://www.it-connect.fr/tuto-installer-et-configurer-opnsense/) pour la suite de la démarche d'installation **Attention il faut tout de même adapter à vos besoins**
---

## 2) Attribution des interfaces (console OPNsense)

Quand OPNsense demande d’assigner :

```
vtnet0 → WAN
vtnet1 → LAN-DMZ

Valider.

---

## 3) Configuration des IP (console)

Configurer selon le document fourni par M.Caille

---

## 4) Configuration DHCP (interface Web)

Se connecter depuis VLAN Admin :  
```
http://@deVotreOpnSense
```
Login :
```
user : root
pass : opnsense
```

Activer DHCP dans :
```
Services → DHCPv4
```

Pools recommandés :
| VLAN | Pool |
|---|---|
| 10 | 100 Adresses|
| 20 | 100 Adresses|
| 30 | 100 Adresses |
| 40 | Aucun DHCP |

---

## 5) Règles Firewall (minimum)

```
LAN-ADMIN :
ALLOW any → any
```

```
LAN-CLIENTS :
ALLOW réseau VLAN20 → any
```

```
LAN-VOIP :
ALLOW réseau VLAN30 → any
```

```
DMZ :
ALLOW DMZ → WAN (HTTP/HTTPS)
```

---

## 6) Tests de validation

| Test | Résultat attendu |
|---|---|
| VLAN10 → Ping 8.8.8.8 | ✅ Internet OK |
| VLAN20 → Ping une IP sur un autre VLAN | ❌ Inter-VLAN interdit |
| VLAN10 → Interface Web FreePBX | ✅ |
| VLAN30 → Téléphone SIP | ✅ |
| DMZ → Internet | ✅ |

---

## ✅ Fin
Votre architecture réseau segmentée est prête à accueillir les services Docker, GLPI, Wazuh et FreePBX.
