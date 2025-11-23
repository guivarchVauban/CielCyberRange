# Objectif

Installer et configurer Wazuh-agent sur OPNSense en passant par le gestionnaire de plugins (Firmware->Plugins)

https://docs.opnsense.org/manual/wazuh-agent.html

# Attention pensez à bien lire les instructions après l'installation du plugin

Une fois l'installation et la configuration terminée, validez la bonne remontée d'informations sur le manager puis : 
- Activer la surveillance des logs du firewall et de l’authentification (SSH, web GUI).
- Activer la vérification de l’intégrité des fichiers critiques (/conf/config.xml, /usr/local/etc/*).

# Bonus
Configurer une réponse active qui bannit automatiquement une IP après trop de connexion ssh sur l'interface du OPNSense (Il faudra auparavant activer le ssh sur opnsense)
