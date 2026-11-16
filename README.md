# Action / LSC Smart Connect → ESPHome (sans cloud)

Procédure **reproduite sur une prise Action/LSC 3202088.1** pour remplacer Tuya par ESPHome et piloter l'appareil en local avec Home Assistant, y compris lorsque l'accès Internet est coupé (objectif à vérifier par un test WAN bloqué).

> **État au 16 novembre 2026** : la prise pilote BK7238 est convertie (étalonnage BL0937 et autonomie WAN toujours à valider). La **multiprise LSC 5225000500 CB3S/BK7231N a également été flashée sous ESPHome** : sauvegarde Tuya vérifiée en double, quatre relais et bouton P10 fonctionnels, LED du bouton confirmée sur **P23**, OTA Wi-Fi réussi. La **dernière V3** a passé la validation indiquée par l'utilisateur, mais son flash et sa recette complète ne sont **pas encore confirmés**. [Retour d'expérience détaillé](docs/10-retour-experience-multiprise-5225000500.md). Ne pas généraliser le brochage à d'autres révisions.

## Matériel et statut

| Appareil | Référence commerciale | Identification | Statut |
| --- | --- | --- | --- |
| Prise avec mesure, pilote | Action **3202088.1** / LSC **5225000301** / EAN **8712879165972** | **BK7238 confirmé par Kickstart**, module T1-2S-NL concordant avec l'exemplaire documenté ; BL0937 | **Flash UnTuyaOS3 puis ESPHome V7 réussis** ; mesure non étalonnée |
| Autre prise avec mesure | LSC **5225000300** | Puce/révision à relever | **Non testée** |
| Multiprise « Multiple TV » | LSC **5225000500**, quatre sorties sans compteur | **CB3S / BK7231N**, relais P14/P6/P7/P8, bouton P10, voyant P23 vérifiés sur cet exemplaire | **ESPHome flashé, OTA et commandes Web fonctionnels** ; dernières vérifications de la V3 et recette complète à terminer |
| Relais porte de garage Wi-Fi/RF à 2 canaux | **HFY-KG21WF+RF**, carte **KG21WF+RF V2.0** | **CB2S / BK7231N** monté verticalement, PCB **LN-CB-2S-V1.0_20230518** ; GPIO réels **non cartographiés** | **C6 retenu (connexion Wi-Fi rapportée)** ; UART CB2S non lu, aucun dump ; [identification](docs/11-identification-ln-cb2s-2026-11-16.md), [affectations C6 candidates](docs/15-garage-esp32c6-cablage-gpio.md) |

## Parcours de conversion (une prise à la fois)

1. [Identifier et préparer la prise](docs/01-preparation.md) : relever références, conserver un accès de secours.
2. [Effectuer la conversion OTA via UnTuyaOS3](docs/02-flash-untuyaos3.md) sur **un matériel compatible confirmé** ; correctif NetworkManager documenté.
3. [Configurer Kickstart et installer ESPHome](docs/03-kickstart-esphome.md), en conservant le type de carte/partitions BK7238 Tuya.
4. [Tester le pilotage local sans Internet](docs/04-validation-locale.md) : relais, bouton, Wi‑Fi, Home Assistant, coupures secteur.
5. [Consulter les incidents et solutions](docs/05-depannage.md) si blocage.
6. [Multiprise 4 voies : qualification et recette](docs/06-multiprise.md), **sans reprendre les GPIO de la prise simple**.
7. [Flash série CB3S : historique et procédure](docs/09-multiprise-uart-esphome.md) ; [retour d’expérience et incidents de développement](docs/10-retour-experience-multiprise-5225000500.md) ; [YAML de référence V3 sans secrets](firmware/5225000500/multiprise-tv-selection-p23.yaml) (**configuration complète encore à éprouver après flash V3**).
8. [Nouvelle carte LN-CB2S : CB2S confirmé visuellement et plan de sauvegarde UART](docs/11-identification-ln-cb2s-2026-11-16.md). La **référence commerciale** et les GPIO réels restent à confirmer avant tout flash.

**Garage — ESP32-C6 choisi :** l'utilisateur rapporte une connexion Wi-Fi réussie sur C6, contrairement au C3. [YAML ESPHome C6 V1 (aucune sortie GPIO)](firmware/hfy-kg21wf-rf/garage-wifi-c6-v1.yaml) et [table GPIO C6 ↔ CB2S, broches de boot à éviter](docs/15-garage-esp32c6-cablage-gpio.md). Le fichier GitHub C6 **n'a pas été flashé/testé** lui-même ; les signaux vers U1 ne sont pas encore identifiés. Ancienne [V1 C3, conservée comme historique](docs/13-garage-esp32c3-supermini-v1.md).

**U1 de la carte RF/portail :** [identification probable Holychip HC89S003A (TSSOP20), brochage et test DAT/CLK/RST](docs/12-hc89s003a-u1-brochage-et-liaison-esp32c3.md). **Ne pas brancher directement l'ESP32-C3** avant mesure de l'alimentation et des signaux de U1.

**Pour reproduire les étapes réseau de l'essai historique**, voir [le correctif lié à UnTuyaOS3](patches/README.md). Il concerne précisément la révision amont `14b56c3` du 29/10/2026 et une interface gérée par NetworkManager ; ne pas le réappliquer aveuglément sur les versions futures.

## Configuration ESPHome

Pour la **multiprise**, lire en priorité [le retour d’expérience du 16/11/2026](docs/10-retour-experience-multiprise-5225000500.md) : il documente l’erreur d’affectation de LED P11/P23, les duplications YAML, les erreurs de scripts et la logique d’un bouton physique unique (appuis bref/long + clignotements). La [V3 publiée](firmware/5225000500/multiprise-tv-selection-p23.yaml) est un exemple **sans valeurs de secrets**, non une preuve de validation de sûreté électrique du montage.

Le [firmware V7 archivé](firmware/5225000301/prise-action-test-v7.yaml) est la **base de référence de la prise pilote**, pas un firmware générique prêt à installer sur toutes les prises. Il utilise `bk72xx: board: generic-bk7238-tuya`, relais P24, bouton P26, LED P9/P11 et BL0937 SEL P1, CF P6, CF1 P8. Les secrets sont uniquement des références `!secret`, sans valeurs.

**ATTENTION métrologie** : sous un sèche-cheveux donné pour 1500 W, l'ancienne configuration affichait environ 234,7 V / 2,425 A / 1547 W, ce qui viole `P ≤ U×I`. Les versions ESPHome 2026.10.2 et 2026.11 bêta ne comptent pas de la même manière certaines impulsions très courtes sur BK72xx. Il faut valider avec un **wattmètre indépendant** et des mesures stabilisées, sans correction empirique arbitraire. L'énergie cumulée n'est pas fiable tant que la puissance ne l'est pas.

## Architecture de pilotage local

`Commande HA locale → API ESPHome sur LAN/VLAN IoT → relais`. **Internet, application Tuya et serveurs Tuya ne doivent pas être nécessaires**, une fois la migration effective. Le Wi‑Fi, le routeur/commutateur local et le serveur Home Assistant restent nécessaires au contrôle depuis HA ; le bouton physique doit rester opérationnel sans ces services.

Le réseau IoT peut être dans un VLAN distinct : autoriser les flux nécessaires entre Home Assistant et les prises (en particulier API ESPHome, typiquement TCP 6053, et découverte mDNS si utilisée). Pour prouver l'indépendance réelle, **bloquer le WAN sans couper le LAN** et exécuter la [recette de validation](docs/04-validation-locale.md). Ne pas déplacer Home Assistant vers le VLAN IoT simplement pour contourner un problème de routage.

## Sécurité et limites

- Réseau **230 V** : ne jamais ouvrir, souder ou mesurer une prise sous tension. La procédure Wi‑Fi a été choisie pour éviter l'ouverture. Risque de « brick », perte de garantie ou besoin de récupération série.
- **Aucune sauvegarde intégrale du firmware d'origine n'avait été obtenue** avant la première conversion OTA. Sauvegarde initiale par UART possible seulement hors tension avec matériel et compétence adaptés.
- Un changement de carte, de version TuyaOS, de bootloader ou de référence peut rendre l'image inadaptée. Vérifier **chaque prise**, y compris celles d'un même lot.
- Ne jamais publier `secrets.yaml`, clés locales/API Tuya, identifiants uniques, exports `devices.json`, adresses MAC privées, captures dévoilant des mots de passe ou firmware original propriétaire.
- Les exemples fournis sont documentaires ; toujours **valider et compiler** localement avant un flash.

## Sources et crédits

- [UnTuyaOS3 (tuya-cloudcutter)](https://github.com/tuya-cloudcutter/UnTuyaOS3), exploit et utilitaire OTA dont dépend la première étape.
- [ESPHome Kickstart (LibreTiny)](https://github.com/libretiny-eu/esphome-kickstart) et [carte generic-bk7238-tuya](https://docs.libretiny.eu/boards/generic-bk7238-tuya/).
- [Retour matériel 3202088.1 / même EAN sur Elektroda](https://www.elektroda.com/rtvforum/topic4132679.html).
- [ESPHome BL0937](https://esphome.io/components/sensor/hlw8012/) et [problème de comptage BK72xx #22391](https://github.com/esphome/esphome/issues/22391).

Projet de documentation communautaire indépendant, non affilié à Action, LSC, Tuya ou ESPHome. Licence du dépôt : [MIT](LICENSE).
