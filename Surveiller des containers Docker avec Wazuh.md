# Objectif

Surveiller les containers GLPI (mariadb et glpi) avec le Wazuh Agent qui tourne sur la debian_docker

https://documentation.wazuh.com/current/user-manual/capabilities/container-security/monitoring-docker.html

** Attention vous devez vérifier la version de python installée et installer pip3 et les dépendances avant de configurer ossec.conf **

Une fois la configuration fonctionnelle : 

## Surveiller les logs des conteneurs

- Configurer l’agent pour lire les logs Docker (/var/lib/docker/containers/.../*.log).

- Faire des tests en générant des erreurs dans GLPI (ex: connexion à la base impossible, login incorrect).

- Vérifier que les alertes remontent dans Wazuh.

## Surveiller l’intégrité des fichiers dans un conteneur

- Choisir des fichiers critiques (config GLPI, .env, fichiers PHP).

- Configurer l’agent pour détecter toute modification de ces fichiers.

- Tester en modifiant un fichier et observer l’alerte.

## Surveiller l’activité des conteneurs

Surveiller la création / suppression / démarrage / arrêt des conteneurs.

Créer une règle Wazuh pour alerter si un conteneur GLPI est arrêté ou redémarré sans raison.
