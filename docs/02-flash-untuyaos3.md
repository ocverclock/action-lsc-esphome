# Étape 2 — Du firmware Tuya à ESPHome Kickstart (OTA Wi‑Fi)

Cette étape **modifie le firmware**. Procédure **réussie le 11 novembre 2026** sur Action 3202088.1 / LSC 5225000301 / EAN 8712879165972, firmware Tuya V1.0.0(1.0.2), MCU 1.0.2.

**Danger :** risque de blocage (« brick »). Aucun dump complet de la prise pilote n'avait été obtenu avant l'essai. Un échec après écriture peut nécessiter une récupération UART, appareil ouvert **hors tension**, par une personne qualifiée. **Ne pas essayer sur la multiprise**.

## 2.1 — Obtenir la version du projet ayant servi au test

```bash
git clone https://github.com/tuya-cloudcutter/UnTuyaOS3.git ~/UnTuyaOS3
cd ~/UnTuyaOS3
git checkout 14b56c3
sudo apt install -y iw isc-dhcp-client
iw dev
./UnTuyaOS3.sh --help
ls -lh custom-firmware/T1/
```

L'expérience historique utilisait la version `14b56c3` du 29/10/2026 et l'image **T1/BK7238** `esphome-kickstart-bk7238-tuya-2026-10-01.ota.ug.bin`. Le format T1 a été vérifié par signature `4d 4d 4d 00`. Cette vérification **ne prouve pas** que l'exploit accepte n'importe quelle prise TuyaOS3.

Vérifier la signature, sans flasher :

```bash
xxd -l 4 -p custom-firmware/T1/esphome-kickstart-bk7238-tuya-2026-10-01.ota.ug.bin
# Résultat de la prise pilote : 4d4d4d00
```

## 2.2 — Correction réseau **spécifique** à l'hôte historique

Avec Linux/NetworkManager et une interface `wls1` déjà connectée à un autre Wi‑Fi, la première tentative échouait sur `DHCP did not assign an address`, **malgré un DHCPACK du SmartLife AP**. Deux causes démontrées :

- l'ancienne adresse IPv4 restait sur `wls1` après que NetworkManager lui avait rendu la main ;
- `dhclient -x` retirait l'adresse fraîchement reçue **avant** que `connect.sh` puisse la lire.

Le correctif validé : vider l'ancienne IPv4 **après association** à SmartLife, conserver le client DHCP durant l'OTA, puis fermer DHCP/vider IPv4 **avant** le retour à NetworkManager. Voir [correctif détaillé](../patches/README.md). **Il n'est pas systématiquement nécessaire sur un autre hôte.**

Dans le test non-flash, on obtenait **une seule adresse 192.168.176.100/24** sur `wls1` avec l'AP à `192.168.176.1`, et le retour au Wi‑Fi normal fonctionnait. Ces IP sont celles de l'AP provisoire Tuya, **pas** l'adresse finale de la prise.

Toujours sauvegarder les scripts modifiés, lancer `bash -n` et `git diff --check`, puis tester la seule connexion **sans lancer `UnTuyaOS3.sh`**, puisque le programme principal passe directement au flash après la connexion.

## 2.3 — Mettre **la prise pilote seule** en mode AP

1. Débrancher toute charge électrique de la prise.
2. Mettre la prise en mode AP Tuya (sur l'exemplaire testé : voyant à clignotement lent ; séquence d'appui selon l'appareil).
3. Vérifier la présence d'un réseau ouvert `SmartLife-XXXX`, en 2,4 GHz. L'essai historique a détecté `SmartLife-D52E` ; ce suffixe **n'est pas universel**.
4. Garder une connexion **Ethernet indépendante** pour l'ordinateur ; le Wi‑Fi dédié sera temporairement détaché du réseau habituel.

## 2.4 — Lancer **une seule** tentative OTA, une fois prêt

```bash
cd ~/UnTuyaOS3
sudo ./UnTuyaOS3.sh \
  --interface wls1 \
  --platform T1 \
  --firmware esphome-kickstart-bk7238-tuya-2026-10-01.ota.ug.bin
```

**Remplacer `wls1` par l'interface réellement identifiée**. N'utiliser `T1` et cette image **que** sur un chipset/bootloader compatible vérifié. Le script principal ne réalise **pas** un simple test de connexion : il exécute la phase `ap-ota.py` et peut écrire la flash automatiquement.

### Journaux attendus pour la prise pilote

```text
Found open network: SmartLife-XXXX
Connected to SmartLife-XXXX (address 192.168.176.100/24)
Attempting to send AP commands
Device schema set, waiting for socket reconfiguration
Attempting to send LAN commands
Uploading [########################################] 100.0%
Reported OTA finished!
Restoring networking ... (NetworkManager)
```

Le transfert archivé portait sur **432292/432292 octets**, avec `Reported OTA finished!`. **Un acquittement d'envoi n'est pas la preuve que le firmware démarre**. Ne pas couper l'alimentation pendant l'envoi et ne pas relancer aveuglément en cas d'incertitude.

## 2.5 — Vérifier le nouveau démarrage

- Rechercher l'AP `kickstart-...` du firmware LibreTiny/ESPHome.
- Se connecter au portail `http://192.168.4.1/` pour lui fournir un réseau Wi‑Fi local.
- Retrouver son adresse sur le routeur puis ouvrir la **page Web Kickstart**.
- Contrôler qu'elle indique **BK7238**, flash 2048 KiB et carte `generic-bk7238-tuya` pour l'exemplaire historique.

**Résultat constaté :** Kickstart ESPHome **2026.9.1** a démarré avec LibreTiny 1.13.0, puce BK7238 et page Web accessible. C'est la preuve de conversion réelle, contrairement au seul message `Reported OTA finished!`.

**Critère de passage :** Kickstart démarré, interface Web accessible, famille BK7238 confirmée. Puis [étape 3](03-kickstart-esphome.md).

## Sources

- [Projet et avertissements UnTuyaOS3](https://github.com/tuya-cloudcutter/UnTuyaOS3)
- [Image Kickstart et portail captif](https://github.com/tuya-cloudcutter/tuya-cloudcutter/blob/main/custom-firmware/README.md)
- [Carte / partitions BK7238 Tuya](https://docs.libretiny.eu/boards/generic-bk7238-tuya/)
