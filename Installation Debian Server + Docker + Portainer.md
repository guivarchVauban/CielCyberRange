# Installation Debian Server + Docker + Portainer
Guide simple et reproductible pour installation en environnement Proxmox

---

## 1) Télécharger l’ISO Debian (Netinst)
Télécharger l’image :  
https://www.debian.org/distrib/

Choisir :  
**Small installation image → AMD64 → `debian-12.x.x-amd64-netinst.iso`**

---

## 2) Création de la VM dans Proxmox
| Paramètre | Valeur recommandée |
|----------|--------------------|
| OS Type | Linux → Debian |
| vCPU | 2 |
| RAM | 8 Go
| Disque | 100 Go (qcow2) |
| Carte réseau | vmbr0 + VLAN Tag selon rôle (souvent VLAN 10 Admin) |

---

## 3) Installation Debian (sans interface graphique)

1. Démarrer sur l’ISO
2. Suivre l’installation (choisir hostname, root, user normal)
3. Lors de **sélection des paquets**, faire :

```
[ ] Debian desktop environment   ← DÉCOCHER
[ ] GNOME
[ ] KDE
[ ] XFCE
[ ] Autres environnements...
[*] SSH server                   ← COCHER
[*] Standard system utilities    ← COCHER
```

4. Finir l’installation et redémarrer
5. Se connecter en SSH (recommandé) :
```
ssh user@IP_DU_SERVEUR
```

---

## 4) Mise à jour initiale
```bash
sudo apt update && sudo apt upgrade -y
```

---

## 5) Installation Docker (méthode officielle)

```bash
curl -fsSL https://get.docker.com | sudo sh
```

Activer Docker au démarrage :
```bash
sudo systemctl enable docker
sudo systemctl start docker
```

Vérifier :
```bash
docker --version
```

---

## 6) Installation de Docker Compose Plugin

```bash
sudo apt install docker-compose-plugin -y
```

Tester :
```bash
docker compose version
```

---

## 7) Installation de Portainer (interface web Docker)

Créer un volume dédié :
```bash
docker volume create portainer_data
```

Lancer Portainer :
```bash
sudo docker run -d   -p 9000:9000 -p 9443:9443   --name portainer   --restart=always   -v /var/run/docker.sock:/var/run/docker.sock   -v portainer_data:/data   portainer/portainer-ce
```

Accès Web :
```
https://IP_DU_SERVEUR:9443
```

Créer le compte admin lors de la première connexion.

---

## 8) (Option recommandé) Désactivation IPv6
```bash
echo "net.ipv6.conf.all.disable_ipv6 = 1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

---

## 9) (Option Wazuh / GLPI) Vérification de la RAM
```bash
free -h
```

Si < 3Go libres → augmenter RAM de la VM.

---

## 10) Résultat final

| Service | Accès |
|--------|-------|
| SSH | `ssh user@IP_DU_SERVEUR` |
| Docker | `docker ps`, `docker compose` |
| Portainer UI | `https://IP:9443` |

Serveur prêt à héberger GLPI, Wazuh, MariaDB, et autres services Docker.

---

