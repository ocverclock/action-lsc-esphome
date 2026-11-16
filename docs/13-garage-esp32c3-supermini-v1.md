# Garage HFY-KG21WF+RF — ESP32-C3 Super Mini : première mise en service (16/11/2026)

**État : YAML V1 publié, NON ENCORE flashé/compilé ni testé sur la Super Mini de l'utilisateur.** Il ne déclare **aucun GPIO de commande**, aucune LED physique, aucun relais, aucun protocole de communication U1.

Fichier de firmware : [garage-wifi-c3-v1.yaml](../firmware/hfy-kg21wf-rf/garage-wifi-c3-v1.yaml).

## But de cette étape

- Amorcer la **Super Mini USB-C** indépendamment de la carte relais : Wi-Fi IoT 2,4 GHz, API ESPHome chiffrée vers Home Assistant, OTA chiffré, interface HTTP locale protégée, AP de secours.
- Recevoir dans HA l'état de connexion, l'adresse IP, le SSID, le RSSI et l'uptime ; prévoir un bouton logiciel de redémarrage volontaire.
- **Ne déclencher aucune action** CH1/CH2, aucun apprentissage RF, aucun signal vers le microcontrôleur Holychip U1 tant que la cartographie des broches et tensions n'est pas vérifiée.

## Paramètres

- Cible : `esp32: board: esp32-c3-devkitm-1`, `variant: ESP32C3`, `framework: esp-idf`.
- Nom réseau : `garage-wifi-c3`.
- SSID et mot de passe Wi-Fi : références **existantes** `!secret wifi_ssid_IoT` et `!secret wifi_password_IoT`.
- Secrets **nouveaux à générer et conserver uniquement localement** :

```yaml
garage_c3_api_key: "CLE_BASE64_32_OCTETS"
garage_c3_ap_password: "MOT_DE_PASSE_AP_SECOURS"
garage_c3_web_password: "MOT_DE_PASSE_WEB"
```

Générer localement la clé base64 32 octets :
```bash
python3 -c 'import os,base64; print(base64.b64encode(os.urandom(32)).decode())'
```

Les mots de passe doivent être distincts ; AP secours minimum 8 caractères. Jamais de valeurs réelles, adresses IP, clés ou mots de passe dans GitHub.

**Compatibilité :** `ota: - platform: esphome; encryption:` nécessite ESPHome **2026.9 ou plus récent**. La clé de chiffrement OTA est la même que `api.encryption.key`. Voir [documentation officielle OTA ESPHome](https://esphome.io/components/ota/esphome/). Si Device Builder est plus ancien, **le mettre à jour** plutôt que de publier une configuration OTA non protégée.

## Ordre des opérations

1. **Ne pas souder encore.** Connecter la Super Mini **seule en USB-C** au PC (aucune autre alimentation ni signal raccordé).
2. Copier YAML dans ESPHome Device Builder, ajouter les secrets privés, lancer **Validate** puis **Install** par USB / téléchargement de l'image. Si une Super Mini nécessite l'entrée manuelle en bootloader via le bouton BOOT + reset/rebranchement, suivre la procédure exacte de son modèle.
3. Vérifier la connexion au Wi-Fi IoT, son IP locale, les logs, l'ajout à HA et la page Web protégée. Confirmer un OTA fonctionnel **avant soudure**.
4. Pour l'intégration à la carte **KG21WF+RF V2.0**, reprendre la cartographie dans [dossier HC89S003A](12-hc89s003a-u1-brochage-et-liaison-esp32c3.md) : déterminer la tension disponible et la consommation de la carte d'origine, puis affecter les broches **uniquement après identification**.
5. Lors du montage, **aucune alimentation secteur ou motorisation**, aucun câblage du 5 V relais sur un GPIO, aucune source 3,3 V connectée simultanément à l'alimentation USB sans prévention du retour de courant. Le 3,3 V logique est nécessaire, mais le régulateur existant peut être sous-dimensionné. Les GPIO de strapping du C3 (GPIO2/8/9) ne sont pas utilisables au hasard en sortie pour les relais. La Super Mini **n'est pas un remplacement mécanique pin-à-pin** du CB2S.
6. **Retirer ou isoler électriquement le CB2S** pour empêcher deux contrôleurs de conduire le même signal. Une déconnexion de GND seule ne suffit pas.
7. Seul le test V1 validé sur banc autorise l'ajout ultérieur des entrées/sorties de commande. Pour un système d'ouverture, les sorties doivent rester non actives au **boot/reset/OTA** ; valider le mode impulsionnel et les protections existantes.

## Sources

- [ESPHome OTA chiffré](https://esphome.io/components/ota/esphome/)
- [Profil ESP32-C3 Super Mini — ESP-IDF](https://www.sudo.is/docs/esphome/boards/esp32c3supermini/)
- [Fiche ESP32-C3](https://documentation.espressif.com/esp32-c3_datasheet_en.html)
