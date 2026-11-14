# Deuxième prise — Surpresseur (14 novembre 2026)

**Statut :** firmware d'origine remplacé par **ESPHome Kickstart démarré**. Firmware définitif personnalisé **non déployé** et test WAN **non encore effectué**.

## Matériel confirmé par l'utilisateur

- Nom futur dans Home Assistant : `Surpresseur` ; identifiant ESPHome : `surpresseur`.
- Même **EAN 8712879165972** que la première prise Action/LSC 3202088.1 / LSC 5225000301.
- Mêmes versions **firmware Tuya principal et MCU** que la prise pilote.
- iMac Linux Ethernet actif, interface Wi-Fi `wls1`, UnTuyaOS3 `14b56c3` avec correctifs NetworkManager/DHCP en place.
- Image de référence T1 BK7238 présente, signature `4d4d4d00` vérifiée.
- AP Tuya détecté ; le flash a envoyé **432292/432292 octets**, mais **sans paquet final de confirmation**.
- Un scan ultérieur a trouvé **`kickstart-bk7238-tuya`** : preuve du démarrage du firmware alternatif.
- L'utilisateur a configuré Kickstart sur le réseau IoT ; une IP locale lui a été attribuée (adresse non publiée dans le dépôt pour éviter de figer des informations de réseau privé).

## Prochaine séquence

1. Vérifier l'ouverture de la page HTTP Kickstart depuis le poste de travail, et la puce/partition `BK7238 / generic-bk7238-tuya`.
2. Créer un **nouvel appareil ESPHome** `surpresseur`, à partir de [`surpresseur-v1.yaml`](../firmware/5225000301/surpresseur-v1.yaml).
3. Ajouter **localement** les quatre nouveaux `!secret` : `surpresseur_api_key`, `surpresseur_ota_password`, `surpresseur_ap_password`, `surpresseur_web_password`. Conserver les noms existants de secrets Wi-Fi, **si ils existent** dans le Device Builder. Générer une nouvelle clé API dans le format ESPHome attendu.
4. Vérifier les SSID configurés : **IoT prioritaire**, autres SSID seulement s'ils sont réellement utilisables et routables. Adapter les entrées `!secret` à l'installation.
5. Faire `Validate`, puis compiler avec ESPHome Device Builder. Aucun flash depuis une configuration non compilée.
6. Depuis **Kickstart**, installer l'artefact **LibreTiny/ESPHome UF2** compatible avec cette carte. **Ne pas** réutiliser l'image `.ota.ug.bin` du premier flash.
7. Vérifier l'API HA, page Web, bouton, relais, Wi-Fi, retour d'état **sans aucune charge branchée**.
8. Tester indépendance WAN ; garder la mesure BL0937 marquée **non étalonnée**.
9. **Avant de rebrancher le surpresseur** : vérifier la puissance nominale **et le courant d'appel du moteur**, la compatibilité du relais de prise avec **les charges inductives**, le comportement à la mise sous tension, et que la coupure de l'alimentation ne compromet ni protection ni sécurité de l'installation. En cas de doute, pilotage par contacteur adapté plutôt que commande directe de la pompe.

**Aucune clé, aucun mot de passe, aucun identifiant unique ni adresse IP locale ne figure dans ce document.**

Voir [la procédure d'installation](03-kickstart-esphome.md), [la recette de test](04-validation-locale.md) et [le suivi de conversion Surpresseur](https://github.com/ocverclock/action-lsc-esphome/issues/5).
