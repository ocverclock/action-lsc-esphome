# ESP32-C3 Super Mini / HFY-KG21WF+RF — connexion UniFi IoT : `Auth Expired` (16/11/2026)

## Journal de démarrage

**Observation utilisateur, ESPHome 2026.10.2 sur ESP32-C3 rev0.4, compilé le 16/11/2026 à 14:47 +0100 :**

- Le firmware V1 passe `setup() finished successfully!` ; le boot, l'initialisation et le scan radio sont opérationnels.
- Le SSID `IoT` est visible avec bon indicateur de réception.
- Connexion au point d'accès (BSSID anonymisé dans cette fiche), puis `Disconnected ssid='IoT' ... reason='Auth Expired'`.
- Plusieurs tentatives, avec priorité de nouvelle tentative décroissante ; **aucune connexion IP/DHCP ni liaison HA confirmée**.
- `Last reset too quick; invoke in 7 restarts` est un avertissement du mode de récupération après redémarrages rapides ; pas la cause montrée par ces logs.
- `Set flash_chip: generic under esp32 > framework > advanced to save IRAM` est un avertissement de configuration mémoire, **pas le motif du rejet Wi-Fi**.

**Interprétation officielle Espressif :** `WIFI_REASON_AUTH_EXPIRE` (code 2) : l'authentification expire ou le point d'accès retourne ce motif ; **ce n'est pas une preuve de mot de passe erroné**. [ESP-IDF reason codes](https://docs.espressif.com/projects/esp-idf/en/v5.2/esp32/api-guides/wifi.html).

## Mise à jour : vérifier d'abord la puissance d'émission du C3 Super Mini

**Correction de priorité après recherche publique ciblée (16/11/2026)** : le profil ESPHome `esp32-c3-devkitm-1` **n'est pas manifestement erroné** ; les journaux de l'utilisateur confirment ESP32-C3 rev0.4, app chargée, `setup() finished successfully!`, et scan Wi-Fi opérationnel. La V1 ne définit **aucun GPIO physique** : il n'existe donc pas d'affectation GPIO qui puisse expliquer ces erreurs d'authentification en l'état.

Un [retour d'expérience Home Assistant du 20 août au 1er octobre 2026](https://community.home-assistant.io/t/esp32-c3-super-mini-wi-fi-auth-expired-no-tx-current-spikes-wroom-32d-works-perfectly/1021889) rapporte **exactement `Auth Expired` sur des ESP32-C3 Super Mini**, avec une configuration pourtant opérationnelle sur d'autres ESP32 :
- L'auteur initial a obtenu une connexion en limitant `wifi.output_power` à **8.5dB** (avec ajout d'un condensateur).
- Un second utilisateur a comparé les puissances sur Super Mini C3 rev0.4 : **20 dB : 0/10 connexions en redémarrage chaud et échecs à froid** ; **11 dB ou 8,5 dB : 30/30 connexions en démarrage à froid** pour chacun. Ces chiffres sont propres à son matériel et **ne garantissent pas notre résultat**.
- La présence de points d'accès au scan démontre la réception, **pas** la bonne qualité de l'émission radio ; l'antenne céramique des Super Mini et certains régulateurs/alimentations sont des causes plausibles.

**Nouveau premier test, avant toute modification d'UniFi :** sur le firmware V1 toujours **USB-C seul**, ajouter dans `wifi:` l'unique ligne `output_power: 11dB` et recompiler/flasher par USB :

```yaml
wifi:
  ssid: !secret wifi_ssid_IoT
  password: !secret wifi_password_IoT
  power_save_mode: NONE
  output_power: 11dB
  # conserver le reste du bloc Wi-Fi actuel, notamment AP de secours
```

Si échec, réessayer à **8.5dB**, puis contrôler câble/source USB, antenne, mot de passe et configuration AP. Tester quelques redémarrages **à froid** si le réseau est obtenu, ainsi qu'à la position finale de la carte (la puissance réduite peut limiter la portée). Ne pas conclure que le réseau UniFi est en cause sans cette comparaison.

**Source de syntaxe :** [ESPHome WiFi output_power](https://esphome.io/components/wifi/). Les profils `esp32-c3-supermini` de tiers et `esp32-c3-devkitm-1` ont tous deux été essayés dans le témoignage ; **changer arbitrairement de board n'est pas le premier correctif**.

## Hypothèses, non encore vérifiées

1. Paramètres SSID IoT incompatibles avec ce module : WPA2/WPA3 mixte, WPA3, PMF obligatoire, fast roaming (802.11r), 802.11v, band steering ou autre option d'authentification avancée.
2. Secret `wifi_password_IoT` erroné ou obsolète : ne pas afficher ni publier sa valeur ; le vérifier sur l'installation privée.
3. Comportement spécifique entre le firmware ESP-IDF/ESPHome C3 et un AP ou une révision de réglage UniFi ; consulter les événements UniFi pour la tentative du nouvel appareil.
4. Problème d'alimentation/USB ou de radio : **moins prioritaire** car le scan montre un SSID bien reçu et un démarrage normal, mais pas totalement exclu.

## Diagnostic conservateur, sans casser le réseau IoT déjà en service

1. **Rester sur la Super Mini seule en USB-C** pour ces essais. Ne pas souder, ne pas câbler les relais.
2. Dans **UniFi Network → Settings → WiFi → IoT**, relever les valeurs actuelles : bande 2,4 GHz ; WPA2 Personal ou WPA2/WPA3 ; PMF ; Fast Roaming ; BSS Transition ; Band Steering. Ne rien modifier avant d'avoir noté l'état.
3. Si WPA3 ou WPA2/WPA3 est activé, tester **WPA2 Personal** uniquement ; si PMF **Required**, essayer **Optional**, puis `Disabled` seulement si besoin sur un SSID WPA2 dédié ou lors du diagnostic. Couper Fast Roaming et BSS Transition si actifs, un paramètre à la fois. Il n'y a aucune raison de dégrader tout un SSID de production qui fonctionne déjà si un SSID de diagnostic isolé peut confirmer la cause.
4. Contrôler **en privé** la correspondance du SSID et du secret de ce nouveau module, sans coller le mot de passe dans le chat/GitHub. Un hotspot temporaire **WPA2 2,4 GHz** ou un SSID temporaire UniFi uniquement pour cet essai permet de départager la configuration de l'ESP et celle de l'AP sans perturber les autres appareils.
5. Lire les événements UniFi au moment des tentatives pour identifier si l'AP rejette l'authentification et le motif ; si le C3 se connecte à un SSID simple, réintroduire ensuite uniquement la configuration nécessaire.
6. Ne pas masquer l'erreur par `fast_connect`, IP statique, réglages VLAN, ou désactivation de l'API : l'erreur survient **avant DHCP et tout trafic IP**.

**Recommandations Ubiquiti pour appareils IoT :** [Unable to Connect to WiFi](https://help.ui.com/hc/en-us/articles/23283660456599-Unable-to-Connect-to-WiFi) et [UniFi WiFi SSID and AP Settings](https://help.ui.com/hc/en-us/articles/32065480092951-UniFi-WiFi-SSID-and-AP-Settings-Overview).

**État :** firmware V1 démarre ; **liaison réseau non validée**, WPA/PMF du SSID IoT encore à confirmer. Aucun changement automatique effectué sur UniFi.
