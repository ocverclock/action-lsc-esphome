# Étape 3 — ESPHome définitif après Kickstart

**Ne pas confondre les formats** : l'image `.ota.ug.bin` T1 est destinée à la **première transition Tuya → Kickstart**, et non à toutes les mises à jour ultérieures. Depuis Kickstart, utiliser le **format ESPHome/LibreTiny approprié (UF2)** ; depuis un ESPHome fonctionnel, l'OTA ESPHome peut être utilisé.

## 3.1 — Première connexion locale

1. Depuis le portail de Kickstart `192.168.4.1`, saisir le SSID et le mot de passe **du Wi‑Fi 2,4 GHz local**.
2. Retrouver la nouvelle IP dans UniFi/le routeur, ou par découverte réseau ; ne pas recopier une IP du journal dans d'autres installations.
3. Ouvrir la page Kickstart ; confirmer `BK7238`, `generic-bk7238-tuya`, et le Web Server actif.
4. Ne **pas** utiliser l'outil GPIO de Kickstart pour actionner des broches au hasard sur un équipement secteur.

## 3.2 — Préparer le fichier YAML

Le [YAML V7 archivé](../firmware/5225000301/prise-action-test-v7.yaml) est un **retour d'expérience complet**, conçu pour la prise pilote. Il conserve :

- `bk72xx: board: generic-bk7238-tuya` (important pour les partitions Tuya BK7238) ;
- relais P24, bouton mécanique P26, LED relais P9, LED d'état Wi‑Fi P11 ;
- `api` Home Assistant chiffrée, `ota` ESPHome, serveur Web avec mot de passe, AP de secours et portail captif ;
- visibilité Wi‑Fi (SSID réel, BSSID, adresse IP, RSSI), redémarrage et sélection de comportement après coupure ;
- lecture **expérimentale** tension/courant/puissance/énergie par BL0937 : SEL P1, CF P6, CF1 P8.

**Personnaliser avant usage** : `substitutions.devicename`, `friendly_name`, références `!secret`, réseaux autorisés, réglages de reprise. Ne pas publier le fichier `secrets.yaml`. Les **quatre** réseaux du V7 reflètent un protocole d'essai, pas une nécessité : **un seul SSID IoT stable est préférable en production** lorsqu'il permet d'atteindre Home Assistant.

Attention : le sélecteur `Demander un réseau Wi‑Fi` fait une **demande** de connexion, ce n'est pas un constat. Seul `wifi_info.ssid` prouve le réseau actif. Changer de VLAN change souvent l'IP et coupe la session Web en cours.

## 3.3 — Exemple de secrets (noms seulement)

Les noms ci-dessous sont ceux de la V7. Créer les valeurs **uniquement dans le fichier local `/config/esphome/secrets.yaml`**. Les chaînes fictives suivantes sont des *emplacements*, pas des identifiants utilisables.

```yaml
prise_action_api_key: "CLE_API_CHIFFREMENT_GENERÉE_LOCALEMENT"
prise_action_ota_password: "MOT_DE_PASSE_OTA_LOCAL"
prise_action_ap_password: "MOT_DE_PASSE_AP_SECOURS_LOCAL"
prise_action_web_password: "MOT_DE_PASSE_WEB_LOCAL"
wifi_unifi: "SSID_UNIFI"
wifi_unifi_password: "SECRET_WIFI_UNIFI"
wifi_ssid_IoT: "SSID_IOT"
wifi_password_IoT: "SECRET_WIFI_IOT"
wifi_ssid_tenda: "SSID_TENDA"
wifi_password_tenda: "SECRET_WIFI_TENDA"
wifi_ssid2: "SSID_SECOURS"
wifi_password2: "SECRET_WIFI_SECOURS"
```

Générer la **clé API au format attendu par ESPHome** dans Device Builder, au lieu de réutiliser littéralement cet exemple.

## 3.4 — Valider, compiler et mettre à jour

1. Ouvrir ESPHome Device Builder dans Home Assistant, créer/importer le YAML et renseigner les `!secret` locaux.
2. **Valider** la configuration avec la version ESPHome choisie, puis **compiler** ; l'analyse YAML seule ne suffit pas.
3. Depuis **Kickstart**, télécharger l'image compilée **`firmware.uf2`**, puis l'envoyer via l'option **OTA Update** de Kickstart. Conserver l'accès réseau pendant l'opération.
4. Après le premier ESPHome fonctionnel, utiliser de préférence **Install → Wireless (OTA)** pour les versions suivantes. Préserver API/OTA et AP de secours.
5. Ne pas couper le secteur pendant l'écriture. Ne pas supposer qu'un `firmware.factory.bin` ou `.ota.ug.bin` est interchangeable avec `.uf2`.

## 3.5 — Vérifier les fonctions **avant** la métrologie

- Démarrage et IP stable après plusieurs minutes.
- Relais ON/OFF depuis page Web et Home Assistant ; **sans charge** au début.
- Bouton physique : appui bref bascule le relais ; appui long non configuré pour reconfiguration réseau en V7.
- P9 suit l'état du relais ; P11 sert à `status_led` (un seul composant doit posséder ce GPIO).
- Comportement au redémarrage : `Dernier état`, `Toujours éteint`, `Toujours allumé` (attention possible transitoire de relais à l'initialisation).
- Wi‑Fi actif lisible et possibilité de retour local via AP de secours.

## 3.6 — Attention au capteur BL0937

La V7 contient `current_resistor: 0.001` et `voltage_divider: 800`, **valeurs de départ présumées**, non mesurées sur la carte. En 2026.10.2, de courtes impulsions inversées peuvent être **comptées deux fois** sur BK72xx ; le correctif testé dans la bêta 2026.11 modifie l'échelle des mesures (voir [issue #22391](https://github.com/esphome/esphome/issues/22391)). Le dernier essai capturé à 1 seconde après redémarrage montrait des mesures **non stabilisées** ; aucune calibration finale validée.

Ne pas corriger `V/A/W` à coups de `multiply` arbitraires. Comparer, sur plusieurs minutes et deux charges non critiques, le wattmètre de référence avec `U`, `I`, `P`, et vérifier `P ≤ U×I` à incertitude près. **Ne pas utiliser les kWh pour facturation ou automatismes dépendants de la puissance tant que les mesures n'ont pas été validées.**

Ensuite : [recette d'acceptation locale](04-validation-locale.md).

## Références

- [ESPHome HLW8012 / BL0937](https://esphome.io/components/sensor/hlw8012/)
- [LibreTiny / BK7238 Tuya](https://docs.libretiny.eu/boards/generic-bk7238-tuya/)
- [ESPHome #18135 — P6/BL0937](https://github.com/esphome/esphome/issues/18135)
- [ESPHome #22391 — compteur d'impulsions](https://github.com/esphome/esphome/issues/22391)
