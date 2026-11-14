# Étape 5 — Incidents rencontrés et solutions éprouvées

Ce tableau reprend les événements **observés** sur l'hôte Linux et la prise Action/LSC 3202088.1 en novembre 2026. Ne pas interpréter un symptôme unique comme une preuve d'incompatibilité matérielle.

| Symptôme | Cause établie / hypothèse | Action prudente |
| --- | --- | --- |
| TinyTuya n'annonce aucune prise | Le Tenda était initialement éteint ; plus tard, la prise se trouvait sur un autre sous-réseau | Vérifier SSID, IP, route et liaison directe. Un scan UDP vide n'est pas un verdict de compatibilité |
| TinyTuya 3.3 : erreur 904 ; 3.4 : 914 | Mauvaise version de protocole pour la prise pilote | Lecture locale réussie en **Tuya 3.5**, avant migration |
| Tuya API : 28841002 | Abonnement IoT Core expiré à l'époque | Ressource réactivée. Cela n'empêche pas l'exploitation UnTuyaOS3 en AP |
| Tuya API matériels : 28841101 | API `/v1.0/iot-03/product/hardwares` non souscrite | Ne pas payer juste pour l'identification ; source matérielle publique ou inspection hors tension |
| AP SmartLife visible, scan UnTuyaOS3 qui boucle | Association non encore prouvée ; parfois `iw scan ... No buffer space available (-105)` puis récupération | Contrôler SSID, mode AP ouvert, interface, état NetworkManager et logs. Ne pas couper un transfert en cours |
| `DHCP did not assign an address` après association | Sur l'hôte historique, `dhclient` recevait **DHCPACK**, mais `dhclient -x` effaçait l'IPv4 avant le contrôle | Utiliser le [correctif testé](../patches/README.md) **pour la révision 14b56c3** |
| Adresse LAN du Tenda et AP Tuya simultanément sur `wls1` | Ancienne IPv4 conservée lors du passage à `unmanaged` | `ip -4 addr flush dev "$iface"` *après association*, avant DHCP, puis restauration propre |
| AP Kickstart disparu après saisie du Wi‑Fi | Normal si l'appareil s'est associé au SSID final | Rechercher dans clients DHCP, mDNS ou voisinage ; ne pas réinitialiser immédiatement |
| Prise « disparue » après flash ESPHome | Connexion à un autre SSID/VLAN, changement d'IP, routage HA absent | Lire le SSID/IP côté routeur ; accès direct Web par nouvelle IP ; vérifier règles VLAN |
| Erreur compilation `TemplateSelect ... no member named state` | Dans ESPHome 2026.10.2, appel C++ incompatible `id(select).state` | Remplacer par **`id(select).current_option()`** comme dans V4.1/V7 |
| BL0937 : tension ou puissance à 0 | GPIO, inversion, mode SEL, bug BK72xx P6 possible (issue #18135) | Vérifier brochage, version logicielle, impulsions, charge résistive ; pas de calibration arbitraire |
| Tension/puissance divisées par deux après beta 2026.11 | Correction de double comptage de certaines impulsions BL0937 inversées sur Beken, issue #22391 | Revoir **ensemble** les paramètres/mesures avec wattmètre ; ne pas masquer par simple `multiply` |
| `ESPHome Logs ... CONNECTION_CLOSED` ponctuel | Session de logs fermée, pas forcément panne du relais | Vérifier uptime, ping, API et retour d'état avant conclusion |

## Séquence de diagnostic non destructive

```bash
# Interface(s) et route vers un appareil connu
ip -br address
iw dev
ip route
# Exemples à adapter aux adresses de votre installation :
ping -c 3 <IP_LOCALE_DE_L_APPAREIL>
ip route get <IP_LOCALE_DE_L_APPAREIL>
```

Avant la conversion Tuya, si le port LAN Tuya est pertinent :

```bash
nc -vz -w 3 <IP_LOCALE_DE_LA_PRISE> 6668
```

Un port 6668 ouvert ne signifie **pas** que le firmware est flashable. Après ESPHome, on attend principalement l'API locale et le serveur Web configuré, **pas** le protocole Tuya.

Pour la découverte AP, utiliser un **autre terminal** uniquement si l'outil principal n'écrit pas déjà le firmware :

```bash
sudo iw dev <INTERFACE_WIFI> scan | grep -i -A 5 'SSID: SmartLife'
```

## Récupération en cas de problème

- **Avant flash, échec de découverte/DHCP** : réparer la liaison Wi‑Fi de l'hôte ; la prise Tuya n'est pas nécessairement modifiée.
- **Upload annoncé terminé, pas d'AP Kickstart** : chercher l'appareil sur DHCP et mDNS puis attendre un redémarrage normal. Ne pas relancer une écriture à l'aveugle.
- **ESPHome démarre mais Wi‑Fi inaccessible** : rechercher tous les SSID configurés ; essayer l'AP de secours si déclenché ; passer par l'IP effective. Préserver l'accès OTA.
- **Bootloader/partitions endommagés** : la récupération peut exiger **UART et un accès matériel hors tension** ; ce projet ne fournit aucune procédure secteur dangereuse de câblage interne. Faire intervenir une personne compétente.
- **Multiprise inconnue** : ne pas appliquer le YAML GPIO de la prise simple, même si le port LAN répond.

## Journaliser utilement

Pour chaque échec, noter la **commande exacte**, le commit de l'outil, la version du firmware, l'étape atteinte (`scan`, `association`, `DHCP`, `ap-ota`, `upload`, `boot`), et seulement les 20 à 30 dernières lignes **assainies**. Ne jamais publier clé, MAC ou IP privée inutile.

## Sources

[UnTuyaOS3](https://github.com/tuya-cloudcutter/UnTuyaOS3) · [ESPHome #18135](https://github.com/esphome/esphome/issues/18135) · [ESPHome #22391](https://github.com/esphome/esphome/issues/22391).
