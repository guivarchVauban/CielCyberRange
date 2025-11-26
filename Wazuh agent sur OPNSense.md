# Objectif

Installer et configurer Wazuh-agent sur OPNSense en passant par le gestionnaire de plugins (Firmware->Plugins)

[Doc OPNSense Wazuh Agent](https://docs.opnsense.org/manual/wazuh-agent.html)

# Attention pensez à bien lire les instructions après l'installation du plugin

Une fois l'installation et la configuration terminée, validez la bonne remontée d'informations sur le manager puis : 
- Activer la surveillance des logs du firewall et de l’authentification (SSH, web GUI).
- Activer la vérification de l’intégrité des fichiers critiques (/conf/config.xml, /usr/local/etc/*).

## Intrusion Detection (Suricata)

Activer la surveillance des Interfaces Wan et Données

[Doc OPNSense Suricata](https://docs.opnsense.org/manual/ips.html)

Sur OPNSense les fichiers relatifs à Suricata se trouvent dans  :

```/var/log/suricata```

- Latest est un lien symbolique vers le dernier fichier de log
- Eve.json contient les alertes relevées par Suricata (C'est ce fichier qui doit être surveillé par Wazuh-agent)
- Stats.log contient des statistiques sur les captures
