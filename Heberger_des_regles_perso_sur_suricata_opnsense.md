Règles Personnalisées via Nginx Docker

Cette méthode garantit que vos règles personnalisées (custom.rules) sont téléchargées par OPNsense sans être écrasées lors des mises à jour du système.
# Étape 1 : Préparation de la Structure et des Fichiers Hôte

Sur votre machine hôte (celle qui exécute Docker) :

Créez la structure de dossiers pour Nginx et vos règles :
        mkdir -p docker_nginx/html/suricata

Créez le fichier de règles (custom.rules) et placez-y votre syntaxe Suricata (par exemple, la règle DHCP) :
    docker_nginx/html/suricata/custom.rules

    (Optionnel) Créez un fichier index.html dans nginx/html/suricata/ pour la documentation web.

# Étape 2 : Création du Fichier docker-compose.yml

Créez le fichier docker-compose.yml (ou modifiez l'existant) pour démarrer un service Nginx et exposer le port 80 de l'hôte :
YAML

version: "3.2"

services:
  # Nouveau service Nginx pour les règles
  nginx-rules:
    image: nginx:stable-alpine 
    container_name: nginx-rules
    hostname: nginx-rules
    ports:
      - "80:80" # Map le port 80 de l'hôte au port 80 du conteneur
    volumes:
      # Mappe votre dossier hôte contenant les règles à la racine web de Nginx
      - ./html:/usr/share/nginx/html:ro
    restart: always

(Si vous aviez GLPI ou un autre service sur le port 80, modifiez-le pour utiliser un autre port, par exemple 8080:80).

Démarrez les conteneurs : 
    docker-compose up -d.
# Étape 3 : Intégration dans OPNsense (Création du Rule Set)

Vous devez "dire" à OPNsense que ce nouveau serveur Nginx est une source de règles.

Créer le fichier de métadonnées XML : Sur l'appliance OPNsense (via SSH/Console), créez le fichier custom.xml :
    /usr/local/opnsense/scripts/suricata/metadata/rules/custom.xml

Contenu du fichier (remplacez VOTRE_IP_HOTE par l'IP de la machine Docker) :

```
    <?xml version="1.0"?>
    <ruleset documentation_url="http://VOTRE_IP_HOTE/suricata/index.html">
      <location url="http://VOTRE_IP_HOTE/suricata/" prefix="Custom"/>
      <files>
        <file description="Règles locales">custom.rules</file>
      </files>
    </ruleset>
```
    Télécharger et Activer dans le GUI :

        Naviguez vers Services → Intrusion Detection → Download.

        L'ensemble de règles Custom apparaît dans la liste.

        Activez-le et cliquez sur Download & Update.

# Étape 4 : Activation sur les Interfaces LAN

Enfin, assurez-vous que l'ensemble de règles est appliqué à l'interface souhaitée.

    Allez à Services → Intrusion Detection → Rules.

    Dans la section Rule cochez et activez l'ensemble de règles Custom.

    Cliquez sur Restart le service
