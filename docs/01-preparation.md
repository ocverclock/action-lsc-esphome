# Étape 1 — Identifier, documenter et préparer

**Périmètre :** prise pilote Action 3202088.1 / LSC 5225000301, EAN 8712879165972, utilisée en novembre 2026. **Ne pas extrapoler** aux variantes 5225000300 ni à la multiprise 5225000500.

## Matériel nécessaire

- Un ordinateur Linux avec Ethernet pour garder l'accès au réseau, et une interface Wi‑Fi 2,4 GHz dédiée à l'AP de la prise (exemple historique : interface `wls1` sur un iMac sous Linux).
- Outils `git`, `iw`, `ip`, `nmcli` si NetworkManager, et `dhclient` ou autre client DHCP compatible ; droits administrateur.
- Un Wi‑Fi 2,4 GHz stable, idéalement le SSID IoT définitif, un accès Home Assistant local, et un moyen de suivre les journaux.
- Une **prise test débranchée de toute charge**. Ne pas brancher de matériel sensible durant les essais.

Installation typique sur Debian/Ubuntu/Zorin :

```bash
sudo apt update
sudo apt install -y git iw iproute2 network-manager isc-dhcp-client
iw dev
iw list | grep -A 12 'Supported interface modes'
ip -br addr
```

Ne pas supposer que les noms d'interface du journal historique sont ceux d'un autre PC.

## Relevés obligatoires avant chaque conversion

| Champ | Prise pilote | À relever pour les suivantes |
| --- | --- | --- |
| Référence étiquette | 3202088.1 | Obligatoire |
| Référence LSC/produit | 5225000301 | Obligatoire |
| EAN | 8712879165972 | Obligatoire |
| Firmware principal | V1.0.0(1.0.2) avant flash | Obligatoire |
| Firmware MCU | V1.0.2 avant flash | Obligatoire |
| Famille matériel | BK7238 confirmé *après* flash Kickstart | **À confirmer avant d'employer une image** |
| GPIO / fonctions | Documentation dans `models/5225000301.md` | À vérifier pour chaque révision |
| Mode AP | `SmartLife-XXXX`, clignotement lent | À observer |

Dans le cas historique, le **protocole local Tuya 3.5** a été lu via TinyTuya (3.3/3.4 échouaient). Cela confirme le dialogue LAN **avant conversion**, pas le modèle exact du microcontrôleur.

Les données obtenues via l'application Tuya ou Tuya Developer ne sont **pas indispensables** au flash UnTuyaOS3. Aucune raison de publier ou de réutiliser des `localKey` dans ce dépôt.

## Garde-fous

1. Vérifier la présence d'un retour d'expérience matériel correspondant *exactement* à la référence et au firmware (pour le modèle pilote : [Elektroda](https://www.elektroda.com/rtvforum/topic4132679.html)).
2. Vérifier si la puce/le bootloader sont compatibles avec l'image prévue ; **un EAN identique ne garantit pas chaque révision de PCB**.
3. Conserver le firmware Tuya tel quel jusqu'au jour de conversion ; éviter les mises à jour imprévues.
4. Prévoir la possibilité d'un échec **sans récupération OTA** (récupération série éventuellement nécessaire après ouverture hors tension).
5. Faire une photo de l'étiquette et une copie privée des informations, pas dans GitHub public.
6. Prévoir les tests finaux : commande locale, redémarrage, bouton, connexion HA, secteur, blocage WAN.

**Critère de passage :** une seule prise pilote identifiée, hôte Linux prêt, radio Wi‑Fi détectée, image de plateforme correcte, risques acceptés.
