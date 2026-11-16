# Nouvelle conversion — module LN-CB2S / carte LN-CB-2S-V1.0_20230518 (16/11/2026)

> **Statut : identification du module en cours, aucune sauvegarde ni écriture flash confirmée sur cet exemplaire.** Cette fiche concerne un **nouvel appareil** à identifier commercialement. Ne pas lui attribuer par défaut les GPIO de la multiprise 5225000500 (CB3S) ni ceux de la prise simple 5225000301 (BK7238).

## Constat matériel rapporté par l'utilisateur

- Inscription de carte : `LN-CB-2S-V1.0_20230518` (la date fait partie du marquage ; elle ne constitue pas une date de fabrication prouvée).
- Inscription sur le module : `LN-CB2S`.
- Montage : petite carte fille montée verticalement, soudée à **90°** sur la carte principale.
- L'utilisateur identifie visuellement le module à la fiche [Tuya CB2S](https://solution.tuya.com/hardware/detail/81001).
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

**Niveau de certitude :** les spécifications sont vérifiées pour le **CB2S officiel**, et la correspondance visuelle est rapportée par l'utilisateur. La sérigraphie `LN-` peut aussi désigner la variante/carte porteuse d'un tiers ; l'identité du SoC de cet exemplaire n'a **pas encore été confirmée par lecture UART**. Le brochage des relais, LED, boutons et capteurs n'est **pas déduit** du brochage du module.

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

1. **Identification sans écriture :** photos nettes recto/verso de la carte fille et de sa connexion à la carte principale, référence produit et éventuelle puce de mesure ; repérer l'antenne, les soudures, les éventuels points de test et MCU secondaire.
2. Vérifier continuité hors tension entre points accessibles et fonctions connues **sans déduire les GPIO** du précédent modèle. Réaliser un relevé de l'ancien état Tuya uniquement si l'appareil est encore opérationnel, sans publier ses secrets.
3. **Sécurité secteur :** débrancher toutes les charges et le 230 V ; vérifier absence de tension dangereuse/résiduelle avant manipulation. Une alimentation de prise peut ne pas être isolée du secteur. **Ne jamais connecter USB-UART/PC à l'appareil alimenté par le secteur.**
4. Vérifier une **alimentation stabilisée 3,3 V** à limitation de courant et un USB-UART configuré en logique **3,3 V** ; ne jamais appliquer 5 V au CB2S. Établir GND commun uniquement dans cette configuration **hors secteur** ; repérer 3V3 et GND à partir des pistes et du datasheet, pas à vue d'œil.
5. Quand le composant et les connexions sont identifiés, lancer **une lecture flash intégrale, non destructive, avec ltchiptool en mode BK7231N** (2 MiB attendu), la répéter et comparer les empreintes SHA-256. Garder les deux binaires **hors du dépôt public** (ils peuvent contenir identifiants Wi-Fi/Tuya, calibrations et éléments propriétaires).
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

## À compléter avant création du YAML matériel

- [ ] Appareil exact (modèle, EAN, nom Tuya) relié à la précédente analyse LAN.
- [ ] Photo du module et des 11 points de connexion, orientation établie.
- [ ] Vérification SoC/port série BK7231N sur l'exemplaire.
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
