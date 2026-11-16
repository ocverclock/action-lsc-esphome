# Nouvelle conversion — module LN-CB2S / carte LN-CB-2S-V1.0_20230518 (16/11/2026)

> **Statut : module CB2S identifié visuellement de façon explicite par l'utilisateur, aucune sauvegarde ni écriture flash confirmée sur cet exemplaire.** La prochaine étape est la sauvegarde série, pas une nouvelle identification théorique du module. Cette fiche concerne un **nouvel appareil** à identifier commercialement. Ne pas lui attribuer par défaut les GPIO de la multiprise 5225000500 (CB3S) ni ceux de la prise simple 5225000301 (BK7238).

## Constat matériel rapporté par l'utilisateur

- Inscription de carte : `LN-CB-2S-V1.0_20230518` (la date fait partie du marquage ; elle ne constitue pas une date de fabrication prouvée).
- Inscription sur le module : `LN-CB2S`.
- Montage : petite carte fille montée verticalement, soudée à **90°** sur la carte principale.
- L'utilisateur confirme que le module visible est **exactement le CB2S** illustré dans la [fiche Tuya](https://solution.tuya.com/hardware/detail/81001) et dans le [guide de flash LibreTiny](https://docs.libretiny.eu/boards/cb2s/#quick-flashing-guide). La carte support porte l'inscription LN-CB-2S, ce qui ne modifie pas la cible ESPHome du module.
- La référence commerciale, le rôle de l'appareil, l'EAN, le firmware Tuya affiché, les pistes, les GPIO affectés et la présence éventuelle d'un MCU auxiliaire restent à relever. Une ancienne analyse LAN est évoquée ; son rattachement à cet exemplaire n'est pas encore établi.
- Le terme « EEPROM » employé oralement décrit ici la **carte module Wi-Fi/MCU** ; il ne faut pas assimiler cette carte entière à une EEPROM externe.

## Identification documentaire : CB2S standard

Selon la [fiche Tuya](https://developer.tuya.com/en/docs/iot/cb2s-module-datasheet?id=Kafgfsa2aaypq) et [LibreTiny](https://docs.libretiny.eu/boards/cb2s/) :

| Propriété | CB2S standard |
| --- | --- |
| SoC | **Beken BK7231N** |
| Mémoire flash | **2 MiB** |
| RAM | **256 KiB** |
| Alimentation logique | **3,0–3,6 V (3,3 V nominal)** |
| Cible ESPHome | `bk72xx: board: cb2s` |
| UART flash | **P10/RX1**, **P11/TX1** ; réinitialisation par **CEN** |
| Partitionnement | Tuya BK7231N, à préserver avec la cible `cb2s` |

**Niveau de certitude :** le **module CB2S est identifié visuellement par l'utilisateur**, et les spécifications/liaisons de flash sont celles du CB2S officiel. La première lecture série confirmera la communication BK7231N et permettra surtout une **sauvegarde**. Le marquage LN-CB-2S de la carte support ne fournit pas les GPIO des relais, LED, boutons et capteurs.

### Brochage numéroté de la fiche officielle Tuya CB2S

| Pin du module | Signal |
| --- | --- |
| 1 | 3V3 |
| 2 | P6 |
| 3 | GND |
| 4 | P7 |
| 5 | RX1 / P10 (entrée UART : vers **TX** adaptateur) |
| 6 | P8 |
| 7 | TX1 / P11 (sortie UART : vers **RX** adaptateur) |
| 8 | ADC / P23 |
| 9 | P24 |
| 10 | CEN (reset, non-alimentation) |
| 11 | P26 |

**Orientation à confirmer visuellement avant tout branchement :** ces numéros proviennent de la documentation du CB2S standard. Ne pas supposer qu'une rangée de soudures sur la carte LN correspond directement à cet ordre, ni qu'une pastille marquée `RX` côté carte mère soit directement accessible sans autre électronique.

## Stratégie de migration prévue

1. **Préparation du câblage :** photographier l'orientation des soudures sur la carte fille CB2S et sa connexion à la carte principale, relever référence produit et éventuelle puce de mesure ; repérer l'antenne, les éventuels points de test et MCU secondaire. L'identification du type de module CB2S n'est plus bloquante.
2. Vérifier continuité hors tension entre points accessibles et fonctions connues **sans déduire les GPIO** du précédent modèle. Réaliser un relevé de l'ancien état Tuya uniquement si l'appareil est encore opérationnel, sans publier ses secrets.
3. **Sécurité secteur :** débrancher toutes les charges et le 230 V ; vérifier absence de tension dangereuse/résiduelle avant manipulation. Une alimentation de prise peut ne pas être isolée du secteur. **Ne jamais connecter USB-UART/PC à l'appareil alimenté par le secteur.**
4. Vérifier une **alimentation stabilisée 3,3 V** à limitation de courant et un USB-UART configuré en logique **3,3 V** ; ne jamais appliquer 5 V au CB2S. Établir GND commun uniquement dans cette configuration **hors secteur** ; repérer 3V3 et GND à partir des pistes et du datasheet, pas à vue d'œil.
5. Quand les **connexions physiques** sont repérées et vérifiées, lancer **une lecture flash intégrale, non destructive, avec ltchiptool en mode BK7231N** (2 MiB attendu), la répéter et comparer les empreintes SHA-256. Garder les deux binaires **hors du dépôt public** (ils peuvent contenir identifiants Wi-Fi/Tuya, calibrations et éléments propriétaires).
6. Vérifier avant toute écriture que la lecture identifie bien la famille attendue, que la sauvegarde est restaurable, que la compilation ESPHome utilise `board: cb2s`, et que le programme ne pilote aucun relais non cartographié.
7. **Flash UART seulement après ces contrôles.** Le firmware `generic-bk7238-tuya` de la prise pilote et le firmware `cb3s` de la multiprise sont **inadaptés** en tant que fichiers à copier directement.
8. Compléter ultérieurement le relevé de GPIO par tests progressifs, sans charge secteur et sans présumer des polarités. Tester les sorties uniquement dans un boîtier reconstitué et sûr, puis le contrôle LAN, Home Assistant et l'indépendance du WAN.

## Séquence de vérification locale (aucune écriture)

```bash
ltchiptool --help
ltchiptool list families
ltchiptool flash read --help
```

Exemple **uniquement après vérification du câblage hors secteur, de la famille et de l'alimentation** :

```bash
ltchiptool flash read -d /dev/ttyUSB0 bk7231n sauvegarde-ln-cb2s-original-1.bin
ltchiptool flash read -d /dev/ttyUSB0 bk7231n sauvegarde-ln-cb2s-original-2.bin
sha256sum sauvegarde-ln-cb2s-original-{1,2}.bin
wc -c sauvegarde-ln-cb2s-original-{1,2}.bin
```

L'absence d'un lien UART réussi ne doit pas conduire à forcer le flash, changer au hasard de famille ou mettre sous tension secteur pendant les essais.


## Incident initial : échec de liaison UART (16/11/2026)

**Observation utilisateur :** alimentation stabilisée 3,3 V jugée correcte, TX et RX câblés ; plusieurs tentatives de lecture `ltchiptool` ont échoué malgré des contacts CEN–GND. Aucun dump réussi ; **message d'erreur exact non encore fourni**.

**Point clarifié :** `CTS` sur l'adaptateur FT232 est le signal de contrôle de flux série, **pas** le reset de la puce. Ne pas le connecter à CEN, ni chercher à piloter CEN via CTS. Le reset est la broche **CEN du CB2S** reliée brièvement à **GND**.

**Procédure non destructive retenue :**
1. Confirmer que l'appareil est complètement débranché du **230 V**, et que l'alimentation logique 3,3 V indépendante et le FT232 partagent **GND**, avec **niveaux UART 3,3 V**.
2. Démarrer la commande de **lecture**, attendre que le programme tente d'établir la liaison (message du type `Connecting` / attente de boot).
3. Faire un **contact bref** CEN vers GND puis relâcher, sans maintenir le reset ; attendre le résultat. Un repère pratique issu d'autres outils BK72xx est **environ 0,25 seconde** (sans précision stricte).
4. Alternative souvent plus simple quand le contact CEN échoue : **couper puis remettre l'alimentation stabilisée 3,3 V** pendant que l'outil essaie de se connecter, sans toucher au secteur ni au câblage USB.
5. Éviter les resets répétés trop rapidement **pendant une négociation déjà commencée**. En cas d'échec persistant, vérifier masse commune, TX/RX croisés, port série libre et réel, tension **sur le module** pendant le reset et stabilité des soudures ; recueillir **les 10 dernières lignes exactes** de l'erreur avant tout nouveau changement.
6. N'effectuer **aucune écriture** tant que la première lecture de 2 MiB n'a pas fonctionné, puis la seconde lecture identique.

**Références techniques :** [LibreTiny CB2S quick flashing guide](https://docs.libretiny.eu/boards/cb2s/#quick-flashing-guide), [LibreTiny BK72xx flashing, reset CEN et alimentation](https://docs.libretiny.eu/docs/platform/beken-72xx/#flashing), [Guide BK7231GUIFlashTool — alternative reset par coupure/alimentation 3,3 V](https://github.com/openshwprojects/BK7231GUIFlashTool#detailed-usage-instructions-bk72xx).

## À compléter avant création du YAML matériel

- [ ] Appareil exact (modèle, EAN, nom Tuya) relié à la précédente analyse LAN.
- [x] Module CB2S identifié visuellement par l'utilisateur et cible LibreTiny `cb2s` sélectionnée.
- [ ] Photo ou repérage électrique fiable des points de connexion, orientation établie.
- [ ] Première connexion et lecture série BK7231N réussies (contrôle technique avant dump).
- [ ] Vérification 3V3/GND hors secteur ; liaison TX/RX documentée.
- [ ] Deux sauvegardes complètes de même SHA-256, stockées en privé.
- [ ] GPIO, niveaux actifs, éventuel MCU de relais/mesure et comportement au boot identifiés.
- [ ] YAML ESPHome `cb2s` validé/compilé, puis flash et recette réussis.
- [ ] Autonomie WAN et intégration HA confirmées.

## Sources

- [Fiche fabricant Tuya CB2S](https://solution.tuya.com/hardware/detail/81001)
- [Datasheet CB2S Tuya : définition des broches](https://developer.tuya.com/en/docs/iot/cb2s-module-datasheet?id=Kafgfsa2aaypq)
- [LibreTiny : cible CB2S et procédure de flash](https://docs.libretiny.eu/boards/cb2s/)
- [ltchiptool](https://docs.libretiny.eu/docs/flashing/tools/ltchiptool/)
