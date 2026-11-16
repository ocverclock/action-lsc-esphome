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

**Observation utilisateur :** alimentation stabilisée 3,3 V jugée correcte, TX et RX câblés ; plusieurs tentatives de lecture `ltchiptool` ont échoué malgré des contacts CEN–GND. Aucun dump réussi. **Message exact désormais fourni :** `TimeoutError: Timed out attempting to link with chip` dans `bk7231tools/serial/linking.py`, ligne 23. Ce message signifie qu'aucune négociation bootloader série n'a abouti ; il ne permet pas, seul, d'incriminer le timing, le firmware ou le chipset.

**Point clarifié :** `CTS` sur l'adaptateur FT232 est le signal de contrôle de flux série, **pas** le reset de la puce. Ne pas le connecter à CEN, ni chercher à piloter CEN via CTS. Le reset est la broche **CEN du CB2S** reliée brièvement à **GND**.

**Procédure non destructive retenue :**
1. Confirmer que l'appareil est complètement débranché du **230 V**, et que l'alimentation logique 3,3 V indépendante et le FT232 partagent **GND**, avec **niveaux UART 3,3 V**.
2. Démarrer la commande de **lecture**, attendre que le programme tente d'établir la liaison (message du type `Connecting` / attente de boot).
3. Faire un **contact bref** CEN vers GND puis relâcher, sans maintenir le reset ; attendre le résultat. Un repère pratique issu d'autres outils BK72xx est **environ 0,25 seconde** (sans précision stricte).
4. Alternative souvent plus simple quand le contact CEN échoue : **couper puis remettre l'alimentation stabilisée 3,3 V** pendant que l'outil essaie de se connecter, sans toucher au secteur ni au câblage USB.
5. Éviter les resets répétés trop rapidement **pendant une négociation déjà commencée**. En cas d'échec persistant, vérifier masse commune, TX/RX croisés, port série libre et réel, tension **sur le module** pendant le reset et stabilité des soudures ; recueillir **les 10 dernières lignes exactes** de l'erreur avant tout nouveau changement.
6. N'effectuer **aucune écriture** tant que la première lecture de 2 MiB n'a pas fonctionné, puis la seconde lecture identique.

**Références techniques :** [LibreTiny CB2S quick flashing guide](https://docs.libretiny.eu/boards/cb2s/#quick-flashing-guide), [LibreTiny BK72xx flashing, reset CEN et alimentation](https://docs.libretiny.eu/docs/platform/beken-72xx/#flashing), [Guide BK7231GUIFlashTool — alternative reset par coupure/alimentation 3,3 V](https://github.com/openshwprojects/BK7231GUIFlashTool#detailed-usage-instructions-bk72xx).


### Diagnostic ciblé après le timeout

**Ne pas lancer de `flash write`.** Vu les multiples tentatives de CEN déjà effectuées, passer à un test qui distingue un défaut du câble/adaptateur d'un défaut côté CB2S :

1. Couper l'alimentation 3,3 V et **déconnecter totalement TX/RX du CB2S** (jamais de secteur).
2. Raccorder **ensemble les deux extrémités des fils TX et RX côté module** (test de boucle locale de toute la liaison), le FT232 restant raccordé au PC.
3. Dans le venv où `pyserial` est installé, lancer :

```bash
python - <<'PY'
import serial
p = serial.Serial('/dev/ttyUSB0', 115200, timeout=1,
                  xonxoff=False, rtscts=False, dsrdtr=False)
p.reset_input_buffer()
p.write(b'CB2S-TEST')
print(repr(p.read(9)))
p.close()
PY
```

**Attention :** la commande ci-dessus doit afficher `b'CB2S-TEST'` si la boucle série fonctionne. Si elle n'affiche rien, vérifier le port, les fils et le FT232 avant tout retour au CB2S. Si elle fonctionne, rebrancher TX/RX croisés sur les véritables **P10/RX1 et P11/TX1**, vérifier tension au pied du module sous charge ainsi que l'absence de collision éventuelle avec un MCU de la carte principale.

**Broches de contrôle :** `CTS` ne sert pas au reset ; certains FT232 exposent aussi `RTS`, qu'un logiciel peut utiliser pour déclencher CEN (optionnel, **uniquement après vérification de la polarité et des niveaux logiques**). Cela n'est pas requis pour une sauvegarde manuelle.

**Résultat de diagnostic confirmé le 16/11 :** boucle locale FT232 et fils réussie : réponse exacte `b'CB2S-TEST'`. Le port USB, l'émission TX, la réception RX et les deux fils (jusqu'à leurs extrémités testées) sont fonctionnels ; cela ne teste pas la soudure ni le raccordement au CB2S.

**Prochain essai unique :** reconnecter FT232 TX → CB2S P10/RX1, FT232 RX ← CB2S P11/TX1, GND commun ; **aucun 230 V**, alimentation stabilisée 3,3 V. Démarrer `ltchiptool flash read` avec le CB2S hors tension 3,3 V, puis alimenter le module **au moment où l'outil attend la liaison** (power-cycle au lieu d'appui CEN). Si timeout persiste, ne pas répéter à l'infini : mesurer 3,3 V directement aux points 3V3/GND du CB2S pendant la tentative et vérifier l'emplacement des pads et l'éventuelle interaction de la carte mère avec P10/P11 ; demander une photo des raccordements.

**Alternative après cet essai :** contact CEN–GND bref pendant l'attente, une tentative contrôlée, puis nouveau diagnostic si échec. Ne rien écrire en flash.

### Nouveau constat : cavalier 5 V / 3,3 V sur FT232

L'utilisateur indique que la lecture `ltchiptool` échoue aussi après mise sous tension 3,3 V au moment de la recherche de connexion. L'adaptateur FT232 dispose d'un **cavalier 5 V / 3,3 V**.

**Important :** le fonctionnement de ce cavalier dépend du schéma de l'adaptateur. Sur plusieurs modules FT232RL, il sélectionne la tension **VCCIO**, donc le niveau haut des signaux logiques TX/RX, et parfois également la tension disponible sur la broche VCC de sortie. On ne peut pas déduire sa fonction exacte sans référence du module ou mesure. Documentation FTDI : https://cdn.sparkfun.com/datasheets/BreakoutBoards/DS_FT232R.pdf

**Contrôle non destructif décidé avant toute nouvelle lecture :**
1. FT232 connecté à l'USB, cavalier placé sur **3,3 V** ; CB2S **non relié aux signaux TX/RX du FT232** ; secteur absent.
2. Au multimètre mesurer **FT232 TX → FT232 GND** au repos : **environ 3,3 V** attendu. Si proche de 5 V, **ne pas reconnecter au CB2S** : identifier la carte FT232, la tension VCCIO et le cavalier.
3. Ne pas relier la **sortie VCC** du FT232 à l'alimentation externe 3,3 V utilisée pour le CB2S ; conserver uniquement le **GND commun** et les signaux TX/RX croisés quand la tension a été validée.
4. Contrôler ensuite la **tension aux broches 3V3 et GND du CB2S pendant la tentative**, et non seulement à la sortie de l'alimentation stabilisée. Si le diagnostic est encore bloqué, demander une photo nette de la carte et des raccordements, et relever la position du cavalier.

**État :** niveau TX et tension au pied du CB2S non encore mesurés ; ne pas multiplier les resets ni supposer que le niveau logique est bon au seul vu de la boucle locale réussie.

### Indice nouveau : LED visible sur la carte lors du raccordement série

Le 16/11, l'utilisateur confirme avoir mesuré **3,3 V sur TX du FT232 au repos**, et signale qu'une **petite LED située sur la carte du module s'allume lorsqu'il raccorde TX et RX** au CB2S. Les tentatives de liaison BK7231N échouent toujours après reset CEN et cycle de l'alimentation externe.

**Interprétation non confirmée :**
- Si la LED s'allume **même lorsque l'alimentation 3,3 V externe du CB2S est coupée**, suspecter une alimentation parasite (« back-powering ») par la ligne UART à l'état haut via les protections d'entrée. Une LED allumée ne prouve pas un démarrage normal ; ne pas continuer à alimenter ainsi.
- Si la LED apparaît **seulement lorsque la véritable alimentation 3,3 V est appliquée**, une liaison UART peut influencer l'état de LED/GPIO sur la carte principale, mais cela ne prouve ni le fonctionnement du bootloader ni une panne matérielle.
- La fiche Tuya du CB2S rappelle notamment que `P10/RX1` ne doit pas être tirée au niveau haut par le circuit hôte en fonctionnement normal ; **ne pas confondre cette recommandation d'intégration avec une interdiction d'utiliser un adaptateur USB-UART pour le flash**.

**Question discriminante avant nouvelle manipulation** : LED allumée avec **alimentation 3,3 V externe coupée** lorsque TX/RX + GND sont raccordés, ou uniquement lorsque le 3,3 V externe est présent ? Ne pas suggérer d'autres resets/flashes jusqu'à clarification. Ensuite, si nécessaire, mesurer le 3,3 V **directement aux pins 1/3 du CB2S** et vérifier le rôle de la LED par inspection des pistes, carte principale possiblement connectée à l'UART.

**Sources :** [Datasheet CB2S Tuya](https://developer.tuya.com/en/docs/iot/cb2s-module-datasheet?id=Kafgfsa2aaypq), [Guide de flash LibreTiny](https://docs.libretiny.eu/boards/cb2s/).

### Photos détaillées de la carte : identification du produit et de la POWER-LED

Le 16/11/2026, l'utilisateur a fourni **cinq photos** de la carte complète, sous plusieurs angles. Conclusions directement étayées par les photos :

- **Carte mère : `KG21WF+RF V2.0`**, contrôleur à deux relais jaunes. Les borniers sont sérigraphiés **ANT / NO / COM / NO / COM / AC-DC** ; ne pas confondre ces entrées et sorties.
- Deux boutons **`LEARN1` et `LEARN2`**, un cavalier de mode présent, des témoins **`POWER-LED`, `WF-LED`, `RF-LED`** et au moins un contrôleur additionnel à nombreuses broches.
- Carte Wi-Fi **`LN-CB-2S-V1.0_20230518`** soudée verticalement, composant CB2S avec blindage métallique.
- La petite LED montrée/nommée par l'utilisateur est **`POWER-LED` située sur la carte mère**, non une LED intégrée au CB2S. Elle ne prouve donc **pas** à elle seule que le microcontrôleur BK7231N est alimenté correctement ou a exécuté son bootloader.
- Sur la carte mère, une rangée de cinq points distincte est marquée **`CLK`, `DAT`, `RST`, `VCC`, `GND`**. **Ne pas l'utiliser comme UART BK7231N sans établir la continuité :** ces points peuvent concerner un autre circuit de contrôle. Il faut utiliser les points réellement connectés aux **P10/RX1** et **P11/TX1** du CB2S selon le brochage officiel et vérifier l'accessibilité derrière les soudures verticales.
- **Notice de matériel à la même sérigraphie et géométrie :** [Universal WiFi Smart Garage Door Controller](https://manuals.plus/m/4a79e6199df1d51af21b25aeac2ff4179b9d2c949d7c71236ab71747f31d50e5). Le matériel commercial voisin est décrit comme un **récepteur Wi-Fi + radio 433,92 MHz à deux relais** pour portail/garage. La notice mentionne plusieurs variantes de tension d'alimentation : ne **pas extrapoler la tension admissible de cet exemplaire** sans son étiquette.
- Le **niveau haut de TX du FT232 a été mesuré à 3,3 V au repos** et la boucle TX/RX du FT232 a réussi ; l'appareil ne répond toujours pas à `ltchiptool`.

**Hypothèse prioritaire à tester :** l'allumage de `POWER-LED` après branchement des fils RX/TX pourrait être lié à une **alimentation parasite de la carte mère par le niveau haut de l'UART**, plutôt qu'à une mise sous tension correcte du CB2S. Ce mécanisme n'est **pas encore prouvé**.

**Test différentiel le plus court, sans flash :**
1. Vérifier que les borniers **AC-DC** et de sortie relais ne reçoivent **aucune alimentation/charge**.
2. Couper l'alimentation externe stabilisée 3,3 V du CB2S et observer si `POWER-LED` s'allume lorsque le FT232 est connecté avec masse commune et TX/RX. Si oui : débrancher immédiatement **TX du FT232 vers RX du CB2S** pour voir si la LED s'éteint. Ne pas prolonger le fonctionnement en alimentation parasite.
3. Avant une autre lecture, mesurer la tension **entre 3V3 et GND directement au module CB2S** avec alimentation stabilisée activée, puis vérifier une **continuité hors tension** des deux fils vers P10/P11 du module. Si l'UART est physiquement relié au contrôleur RF de la carte mère, une interaction/collision reste possible.

**Décision :** pas de `flash write`, pas de cavalier RTS/CTS/CEN improvisé et pas de configuration GPIO ESPHome tant que l'origine des `TimeoutError` n'est pas expliquée.


### 16/11 — masse du module isolée de la carte mère, nouvelle tentative infructueuse

**Observation utilisateur :** après avoir interrompu le **contact GND entre le module CB2S et sa carte mère**, la `POWER-LED` de la carte mère ne s'allume plus lorsque le FT232 est connecté. La **lecture de la flash BK7231N échoue toujours**.

**Interprétation :** l'extinction de cette LED montre seulement que son chemin de courant a été modifié ; elle ne prouve ni un démarrage correct du CB2S ni la disparition de toute alimentation parasite par d'autres GPIO. Le test de boucle locale FT232 était bon et son TX au repos mesuré à ~3,3 V.

**Point critique avant tout autre essai UART :** il faut une référence **GND commune sur le module** entre 1) borne négative de l'alimentation 3,3 V externe, 2) GND du FT232 et 3) **GND/broche 3 du CB2S**. La séparation GND du module et de la carte mère ne doit pas rompre ce triangle. **Ne pas alimenter et tester un montage dont le module n'a pas sa masse commune avec l'alimentation/le FT232.** Si la masse du module n'est plus fiable, suspendre le test.

**Contrôle prioritaire demandé :** alimentations secteur/bornier AC-DC toutes absentes ; alimentation externe 3,3 V seule active ; relever au multimètre la tension **directement entre la broche 1 (3V3) et la broche 3 (GND) du CB2S**, sous cette alimentation. Résultat attendu ~3,3 V ; **ne pas inférer la tension du module d'une LED sur la carte mère ou d'une mesure à la sortie du régulateur**. Pour éviter un court-circuit, mesurer avec pointes isolées et accès fiable ; interrompre si accès dangereux.

**Précaution de conception :** sur une carte mère multi-circuits, couper seulement GND en laissant d'autres signaux interconnectés peut créer des retours parasites via GPIO ; ne pas considérer cet isolement partiel comme une solution validée. Relever exactement quelle liaison physique a été interrompue avant de définir une procédure stable.

**Aucun flash ni effacement réalisé.**

### Précision utilisateur sur l'isolement de masse

L'utilisateur précise que la déconnexion de **GND entre le CB2S et la carte principale** était **volontaire** : il souhaite alimenter et connecter **uniquement le CB2S** pour réaliser la sauvegarde UART. Ne pas présenter cette action comme une perte accidentelle de masse.

**Schéma voulu (à vérifier électriquement)** :
- CB2S 3V3 vers +3,3 V de l'alimentation stabilisée ;
- CB2S GND vers borne négative de l'alimentation stabilisée **et** GND FT232 ;
- CB2S P10/RX1 ← FT232 TX ; CB2S P11/TX1 → FT232 RX ;
- CB2S CEN normalement libre, contact bref GND uniquement si reset demandé ;
- carte mère sans alimentation sur ses borniers AC-DC.

**Réserve importante :** le module reste physiquement soudé à la carte principale sur d'autres broches ; **couper seulement GND n'assure pas une véritable isolation** et peut permettre des retours par GPIO/diodes de protection. Ne pas conseiller de couper d'autres pistes au hasard. Mesurer d'abord la tension réelle entre 3V3 et GND **au niveau du CB2S** ; si elle est correcte et stable, identifier les points UART réels et les liaisons résiduelles avec la carte principale avant d'autres tentatives.

## Recherche de retours d'expérience publics sur les timeouts CB2S/BK7231N (16/11/2026)

**Question de départ :** pourquoi `ltchiptool flash read -d /dev/ttyUSB0 bk7231n ...` échoue-t-il avec `TimeoutError: Timed out attempting to link with chip` alors que la boucle FT232 est OK, TX au repos est mesuré à 3,3 V, les resets CEN et les cycles 3,3 V ont été tentés et l'utilisateur a déconnecté le **GND entre module et carte mère**, sans succès ?

### Cas publics recoupés

| Source | Problème observé | Résultat | Enseignement pour KG21WF+RF |
| --- | --- | --- | --- |
| [Elektroda 3951003, 21–26/01/2023](https://www.elektroda.com/rtvforum/topic3951003.html) | CB2S/BK7231N ne négocie pas UART sur PCB à relais ; RX/TX utilisés par la logique du relais | **Résolu** par dessoudage et programmation de la carte CB2S seule, succès confirmé par l'auteur au post #10 | **Très pertinent :** couper seulement GND de la carte mère ne libère pas RX1/TX1, encore reliés aux autres circuits |
| [Elektroda / KMC 30153, 24/02/2023](https://www.elektroda.com/news/news3960117.html) | ~60 tentatives série infructueuses ; TX1 du CB2S maintenu haut par un bouton | Conversion **OTA** réussie ; le correctif UART (isoler le bouton) est expliqué mais **non testé** dans cet article | Les broches peuvent être bloquées électriquement même quand l'UART FT232 fonctionne seul |
| [Elektroda 4024088, 26–27/12/2023](https://www.elektroda.com/rtvforum/topic4024088.html) | Module CB2S dessoudé ne répond pas ; sérigraphie trompeuse 3V3/GND | **Résolu** après correction du brochage, premier essai réussi | Vérifier les véritables points 3V3/GND et P10/P11 au multimètre, ne pas se fier à une LED de carte mère |
| [Elektroda 3977701, 13/05/2023](https://www.elektroda.com/rtvforum/topic3977701.html) | Variante CB2S dont RX est relié à GND sur PCB, empêche l'accès UART | L'auteur décrit avoir séparé **la soudure de RX** vers la carte mère à la tresse, sans déposer tout le module | Une **isolation ciblée de RX1/P10** peut suffire si une liaison à la carte mère est démontrée ; ne pas sectionner au hasard |
| [LibreTiny libretiny #181](https://github.com/libretiny-eu/libretiny/issues/181) | Timeout sous `ltchiptool` ; outils série se comportent différemment | `bk7231tools` a fonctionné sur un montage de l'auteur, qui était un **autre Beken** ; question du port `ttyUSB0` / `ttyUSB1` soulevée | Option secondaire **une fois le câblage validé**, pas une preuve d'incompatibilité CB2S avec ltchiptool |
| [BK7231GUIFlashTool README](https://github.com/openshwprojects/BK7231GUIFlashTool#detailed-usage-instructions-bk72xx) | Synchronisation bootloader délicate | Recommande reset CEN ~0,25 s ou cycle d'alimentation, signaux UART 3,3 V | Déjà essayé dans notre montage, priorité désormais faible |
| [LibreTiny CB2S officiel](https://docs.libretiny.eu/boards/cb2s/#quick-flashing-guide) | Procédure et pinout du CB2S | **P10/RX1 ← TX FT232 ; P11/TX1 → RX FT232 ; CEN reset ; BK7231N ; 2 MiB** | Confirme le choix de plateforme, **pas le câblage des circuits de la carte mère** |

### Conclusion applicable à notre carte

La première hypothèse à contrôler est **une collision ou un biais électrique sur P10/RX1 ou P11/TX1 entre le CB2S et l'autre électronique de la KG21WF+RF** (microcontrôleur RF, boutons, résistances/capacités), **plus vraisemblable qu'un problème de timing** compte tenu des essais déjà faits. Ce n'est qu'une hypothèse, pas une panne démontrée.

**Protocole de diagnostic conseillé sans écriture :**

1. Tous borniers AC-DC sans alimentation, alimentation externe 3,3 V du module et USB FT232 débranchés. Photographier les points soudés du CB2S, puis identifier physiquement **P10/RX1 et P11/TX1** sur la carte fille **selon le pinout LibreTiny**, à l'aide d'une vérification de continuité des pistes.
2. Vérifier **la masse commune CB2S ↔ source 3,3 V ↔ FT232**, ainsi que 3,3 V **directement entre les points VCC et GND du CB2S**. Le retrait de GND **vers la carte mère** n'a pas la même utilité que le retrait d'une connexion RX1 ou TX1 ; il peut au contraire créer des retours indésirables via les signaux restant reliés à la carte.
3. Hors tension, contrôler à l'ohmmètre si **P10/RX1 est en court-circuit franc vers GND**, si P11/TX1 est associé à une charge directe suspecte et si ces deux lignes rejoignent la logique de la carte mère. Mesures au multimètre seules non suffisantes pour exclure toutes les interférences.
4. **Si une connexion perturbatrice est établie**, privilégier une **déconnexion réversible des seules soudures de P10/RX1 et/ou P11/TX1 vers la carte mère**, par un intervenant compétent ; maintenir le GND de référence du module et de l'alimentation et éviter de créer un module flottant sur d'autres GPIO. Ne pas couper des pistes au hasard ni transférer le brochage d'un autre CB2S.
5. Si les soudures sont inaccessibles, évaluer le dessoudage de la **carte fille entière** plutôt qu'un isolement partiel GND sur une carte complexe. Sécurité : **aucune alimentation secteur** pendant cette intervention ; vérifier absence de charge résiduelle dans les condensateurs.
6. Une fois l'UART réellement libéré et les points validés, réessayer **une lecture seule** `ltchiptool` à 115200 puis une seconde sauvegarde identique. **Ne pas** utiliser l'action BK7231GUIFlashTool « Backup and flash new » qui **écrit automatiquement** après la sauvegarde.

**Ordre de priorité :** vérification physique des points et des liaisons P10/P11 > bon GND/3,3 V au module > libération propre des pistes RX/TX si nécessaire > essai de lecture avec autre utilitaire > essais de reset supplémentaires.

**Interdit à ce stade :** `flash write`, effacement sans sauvegarde, exposition des clés Tuya, branchement USB/FT232 sur un appareil relié au 230 V.


### Observation utilisateur : RX1 et TX1 situés vers le haut de sa carte montée verticalement

L'utilisateur remarque le 16/11 que **les connexions RX1 et TX1 se trouvent sur le haut de sa carte**, d'après son orientation matérielle. Ceci peut expliquer pourquoi les précédents essais ne communiquaient pas, **si les fils avaient été raccordés à d'autres points**. La position « haut » est relative au montage : se fonder sur les étiquettes et la continuité, **pas** sur un repère spatial seul.

**Brochage officiel de la puce/module CB2S Tuya** :
- **RX1 = P10, pin 5 du CB2S** : à relier au **TX du FT232** ;
- **TX1 = P11, pin 7 du CB2S** : à relier au **RX du FT232** ;
- **TX2/RX2** sont des points **distincts** et ne doivent pas servir de remplacement pour la liaison de flash UART1 ;
- le FT232, l'alimentation 3,3 V et le **GND du CB2S** doivent partager une masse de référence même si certaines connexions avec la carte mère ont été séparées.

**Prochaine action :** couper toutes les alimentations et le FT232, identifier positivement les pastilles **RX1/TX1** réellement présentes sur la carte (en cas de doute : continuité hors tension), reconnecter TX/RX **croisés** à ces seules pastilles, vérifier le 3,3 V au module, puis refaire une **lecture seule** avec `ltchiptool` et reset CEN. Il ne faut pas supposer que la simple identification de la bonne face garantit le succès ; les autres liaisons à la carte mère peuvent encore charger UART1.

**Références :** [LibreTiny CB2S, pinout et quick flashing guide](https://docs.libretiny.eu/boards/cb2s/#quick-flashing-guide), [Tuya CB2S, pin definition](https://developer.tuya.com/en/docs/iot/cb2s-module-datasheet?id=Kafgfsa2aaypq).


## Étude d'alternative : remplacer le CB2S par un module ESP32-C3 compatible (16/11/2026)

### Identité commerciale retrouvée dans l'inventaire Tuya précédent

- Appareil : **« Relais porte garage »** ;
- Produit Tuya : **« WIFI&RF smart 2CH switch »** ;
- Modèle : **HFY-KG21WF+RF** ;
- `product_id` de **modèle** : `2liqjx62ebuauimx` (ne pas publier de localKey ni d'identifiant unique d'appareil) ;
- DPs recensés avant ouverture : **1, 2, 9, 10, 41**. Attribution **1/2 aux deux relais probable, mais non vérifiée ici** ; ne pas en déduire les broches GPIO ;
- Sérigraphie carte : **KG21WF+RF V2.0** ; Wi-Fi vertical CB2S ; deux relais et réception RF indépendante d'apparence.
- La [notice d'un contrôleur à la même sérigraphie](https://manuals.plus/m/4a79e6199df1d51af21b25aeac2ff4179b9d2c949d7c71236ab71747f31d50e5) montre des boutons LEARN1/2 et deux contacts de relais à sec. Le catalogue liste plusieurs variantes d'**entrée AC/DC**, dont certaines basse tension et d'autres secteur ; **vérifier la plaque réelle** avant alimentation de la carte (ne pas injecter 230 V sur la base d'une notice générale).

### Fonctions de broches standard CB2S (confirmées par Tuya, pas le câblage de KG21WF+RF)

| CB2S pad | Fonction physique | Usage sur KG21WF+RF |
| --- | --- | --- |
| 1 : 3V3 | 3,3 V | rail d'alimentation module |
| 2 : P6 | GPIO | non cartographié |
| 3 : GND | masse | référence 0 V commune |
| 4 : P7 | GPIO | non cartographié |
| 5 : RX1/P10 | UART1 entrée | destination sur carte mère non cartographiée |
| 6 : P8 | GPIO | non cartographié |
| 7 : TX1/P11 | UART1 sortie | destination sur carte mère non cartographiée |
| 8 : ADC/P23 | ADC ou GPIO | non cartographié |
| 9 : P24 | GPIO | non cartographié |
| 10 : CEN | enable/reset | commande de reset |
| 11 : P26 | GPIO | non cartographié |

**On ne sait pas encore** si relais CH1/CH2 sont pilotés directement par les GPIO du CB2S, via un MCU de la carte mère en UART (protocole Tuya MCU) ou par une logique intermédiaire. Les deux boutons LEARN sont probablement gérés au moins partiellement par le circuit RF, mais **cela reste à tester**. Une nouvelle puce n'apporte pas d'emblée le mappage GPIO.

### Modules de remplacement documentés publiquement

- **[ESP8685-WROOM-03-H4](https://templates.blakadder.com/ESP8685-WROOM-03.html)** : module Espressif **famille ESP32-C3**, flash **4 MiB** ; explicitement référencé comme remplaçant de CB2S. Exemple réel de [CB2S remplacé pour ESPHome](https://devices.control-j.com/devices/miboxer-tri-c1wr-dimmer-module/) ; GPIO et protocole à adapter.
- **[WT32C3-01N](https://templates.blakadder.com/WT32C3-01N.html)** : autre module **ESP32-C3 de remplacement CB2S** cité. Préférer des modules correspondant exactement au **format de pastilles CB2S** ; une carte de développement ESP32-C3 « classique » n'est pas directement compatible mécaniquement.
- Dans cet exemplaire, le CB2S semble fixé sur une **petite carte verticale `LN-CB-2S-V1.0_20230518`**, elle-même soudée à la carte mère. Vérifier le **support** et les surfaces de soudure avant de déclarer un remplacement `drop-in` sur la carte principale.

**Correspondance *théorique de position de pastilles* CB2S → ESP8685-WROOM-03**, issue de la confrontation des deux *pinouts de module standard* (à vérifier **sur le support** au multimètre avant toute soudure, et non assimilable à un mappage fonctions relais) :

| CB2S | ESP8685-WROOM-03 |
| --- | --- |
| 3V3 | 3V3 |
| P6 | GPIO3 |
| GND | GND |
| P7 | GPIO7 |
| RX1/P10 | RX / GPIO20 |
| P8 | GPIO6 |
| TX1/P11 | TX / GPIO21 |
| ADC/P23 | GPIO1 |
| P24 | GPIO5 |
| CEN | EN |
| P26 | GPIO4 |

**Attention :** cette table suppose la bonne orientation sur une implantation de modules compatibles **CB2S/TYWE2S**, pas un câblage direct des numéros de pin arbitraires d'une carte de développement ESP32-C3. La carte support LN-CB2S peut imposer un adaptateur/reprise de soudures spécifique. Toujours valider les rails 3,3 V/GND et l'absence de court-circuit avant mise sous tension. L'ESP32-C3 doit être **préflashé avant montage** avec un firmware minimal qui laisse les sorties relais en état sûr ; ne pas préprogrammer un GPIO de relais en sortie sans identification et polarité vérifiées.

### Décision prudente

1. **Ne pas sacrifier tout de suite le CB2S** ni continuer à laisser GND du module isolé de la carte mère alors que d'autres GPIO restent connectés : stopper les essais électriques avec isolement partiel non maîtrisé.
2. Identifier au multimètre hors tension les **connexions réelles** entre broches P6/P7/P8/P23/P24/P26 du CB2S, transistors de commande relais et/ou microcontrôleur principal ; photographier la face des soudures pour tracer les nets. Les points `CLK/DAT/RST/VCC/GND` sur la carte mère ne sont pas démontrés être des ports UART CB2S.
3. Déterminer si les boutons RF agissent **sans CB2S** ou exigent un retour MCU Wi-Fi ; ne pas supposer que les télécommandes survivront à un échange de module.
4. Si des relais sont gérés par MCU via UART, choisir firmware ESPHome avec `tuya:` et UART **seulement après identification du protocole et des DP** ; sinon configurer les GPIO directement. Les DPs 1/2 ne désignent pas les broches physiques.
5. Préparer et **flasher sur banc** un ESP8685-WROOM-03-H4 sans connecter les relais, tester le boot/Wi-Fi/Home Assistant et la compatibilité du courant disponible sur 3,3 V avant de dessouder/installer. La suppression du CB2S sans sauvegarde d'origine rend le retour Tuya difficile.
6. Tester mécaniquement la fixation verticale, son antenne et sa distance aux autres cartes ; puis recette deux sorties, mode momentané/bistable, sécurité au redémarrage et apprentissage RF, **sans appareil motorisé connecté au premier test**.

**Sources :** [Tuya CB2S pin definition](https://developer.tuya.com/en/docs/iot/cb2s-module-datasheet?id=Kafgfsa2aaypq), [ESP8685-WROOM-03 datasheet HTML](https://documentation.espressif.com/esp8685-wroom-03_datasheet_en.html), [Blakadder ESP8685-WROOM-03](https://templates.blakadder.com/ESP8685-WROOM-03.html), [ESPHome Devices exemple pratique](https://devices.control-j.com/devices/miboxer-tri-c1wr-dimmer-module/).


## 16/11 — précision : code JCYD02 94V-0 et **ESP32-C3 Super Mini** envisagée

L'utilisateur a relevé au dos de la petite carte **`JCYD02 94V-0`**. **Ce marquage n'identifie pas une EEPROM ni un SoC.** Un rapport de test public attribue la référence de circuit imprimé **JCYD02** à **Shenzhen Jinchuangyida Electronic Co., Ltd**, avec une classification **V-0** selon UL 94/UL 796 ; `94V-0` désigne la **classe d'inflammabilité du support PCB**, pas un firmware ni un pinout. Le fait de lire ce code sur l'exemplaire ne démontre pas à lui seul quel circuit électronique est installé. [Rapport citant JCYD02](https://img.yfisher.com/m0/1679645946780-pe-1000-pse.pdf) ; [définition UL 94 V-0](https://svtronics.com/wp-content/uploads/2025/09/PCB_Basics_for_Buyers-Final.pdf).

**Correction de notre proposition de remplacement :** l'utilisateur souhaite employer **une carte de développement ESP32-C3 Super Mini**, **pas** le module ESP8685-WROOM-03-H4 proposé précédemment. Le Super Mini est une **carte complète USB-C avec régulateur** et un brochage de type header 2×8 ; elle **n'est pas compatible mécaniquement ni broche à broche** avec le module Tuya CB2S. La proposition d'ESP8685 plus haut demeure une *alternative d'étude*, mais **n'est pas la solution voulue ici**.

### Broches accessibles typiques d'un ESP32-C3 Super Mini

- Puissance : `5V`, `GND`, `3V3` ;
- GPIO accessibles : **0 à 10, 20 (UART RX) et 21 (UART TX)** ;
- **GPIO2, GPIO8, GPIO9** : strapping de démarrage, à éviter sur relais tant que les niveaux de démarrage ne sont pas maîtrisés ; GPIO8 porte souvent une LED, GPIO9 le bouton BOOT ;
- **GPIO18 et GPIO19** : USB natif, généralement **non exposés** sur le connecteur 2×8 ;
- Niveaux des entrées/sorties **3,3 V uniquement**, jamais brancher directement une commande 5 V/12 V/secteur aux GPIO.

**Alimentation :** sur beaucoup de Super Mini, la broche `3V3` est le rail derrière le régulateur et peut être utilisée pour une source **3,3 V régulée correctement dimensionnée** sur un montage qui le permet ; il faut contrôler la révision exacte de la carte et empêcher l'alimentation simultanée via USB et externe sans gestion de retour de courant. La broche `5V` n'est pas une entrée 3,3 V. Pour le premier flash ESPHome, privilégier **USB-C seul**, Super Mini déconnectée de toute électronique de relais.

### Méthode de conversion retenue pour HFY-KG21WF+RF

1. **Arrêter les essais de lecture du CB2S** en isolement partiel dangereux (GND coupé mais autres signaux encore soudés), jusqu'à maîtrise du câblage. Ne plus traiter cette panne de lecture comme un obstacle absolu à l'architecture de remplacement, mais conserver que le firmware original **n'est pas sauvegardé**.
2. **Relever le câblage réel** de la carte KG21WF+RF au multimètre **hors tension** : P6/P7/P8/P23/P24/P26/RX1/TX1 vers relais, transistors de puissance, LED Wi-Fi, MCU/RF et boutons. Les contacts NO/COM ne sont pas des entrées GPIO.
3. Distinguer commande GPIO directe des relais **via leur étage transistor** d'un éventuel **protocole UART Tuya MCU** ; le composant 16 broches et la carte RF imposent de ne pas affirmer l'architecture sans traçage.
4. **Préprogrammer ESP32-C3 Super Mini en USB-C** avec ESPHome, sans GPIO de sortie affecté aux relais avant d'avoir déterminé les polarités, et sans activer de moteurs.
5. Déposer proprement le CB2S (ou assurer une **déconnexion électrique complète et documentée** de ses sorties) avant de raccorder les signaux à la Super Mini. Ne pas laisser deux microcontrôleurs piloter les mêmes lignes ; ne pas compter sur un simple retrait de GND pour isoler le module.
6. Raccorder seulement les **signaux logiques 3,3 V vérifiés** et le GND de référence, avec alimentation dimensionnée, selon une table *fonction carte KG21WF+RF → GPIO ESP32-C3 Super Mini* à créer. **Aucune correspondance de GPIO n'est à inventer à partir du pinout du CB2S.**
7. Faire une recette sans charge moteur : démarrage sûr, chaque contact sec, retour HA, reconnexion Wi-Fi, coupure/reprise d'alimentation, mode impulsionnel s'il commande un portail, et conservation des fonctions RF si avérées. Ne pas connecter une motorisation tant que les états parasites au boot sont exclus.

**Références :** [brochage ESP32-C3 Super Mini](https://chiptron.eu/pinouts/esp32-c3-super-mini-pinout/), [schéma ESP32-C3 Super Mini](https://smartstream-electronics.co.uk/assets/datasheets/ESP32-C3-SuperMini.pdf), [datasheet officiel Tuya CB2S](https://developer.tuya.com/en/docs/iot/cb2s-module-datasheet?id=Kafgfsa2aaypq).

**Statut :** compatibilité conceptuelle confirmée ; remplacement **non réalisé**, GPIO des relais/RF inconnus, comportement au démarrage non testé. Les coupures/soudures doivent se faire totalement hors tension, secteur et USB débranchés.


## Analyse photographique détaillée de la carte KG21WF+RF V2.0 (16/11/2026)

**Nouvelles vues fournies :** quatre photographies supplémentaires de la **face composants et de la face soudures** (prises des deux orientations). Elles confirment :

| Élément visible | Lecture confirmée sur les photos | Conséquence |
|---|---|---|
| Carte mère | `KG21WF+RF V2.0` | Distinguer de la carte fille `LN-CB-2S-V1.0_20230518` |
| Relais CH1/CH2 | **Deux HUI KE HK4100F-DC5V-SHG**, bobine **5 V DC** (marquage sur le relais) | **Ne jamais alimenter les bobines directement par un GPIO de l'ESP32-C3**. La commande doit passer par l'étage transistor / la logique existante |
| Sorties | Borniers **CH1 NO/COM** et **CH2 NO/COM** : contacts normalement ouverts et communs | Séparer les contacts secs des circuits basse tension |
| Entrée | Bornier **IN AC-DC** avec étage de redressement/régulation visible | La valeur nominale admissible d'entrée n'est **pas vérifiée** sur cet exemplaire ; ne pas injecter du 230 V ni supposer que cette borne est du 5 V |
| Radio | Petit module RF distinct et borne `ANT` | Présence d'une fonction RF séparée ; autonomie des télécommandes **non encore prouvée** |
| Logique | Circuit **U1 à 28 broches environ**, résistances et plusieurs boîtiers SOT-23 | Probable traitement/commande des deux sorties ; **identité et protocole non relevés** |
| Test du contrôleur | 5 pastilles **GND / VCC / RST / DAT / CLK** voisines de U1 | **Ne pas traiter comme RX/TX UART** : évoquent un connecteur programmation/test dédié au contrôleur de la carte |
| Témoins | `POWER-LED`, `RF-LED`, `WF-LED` | `POWER-LED` signale la carte mère, pas nécessairement le démarrage du CB2S |
| Face soudures | Zone de **rangées de contacts du module vertical** et pistes fines cheminant vers la logique | Photos utiles pour orienter les tests de continuité, **insuffisantes pour attribuer les GPIO exacts** |

### Question d'architecture à résoudre

**Hypothèse A — contrôle direct :** le CB2S utilise des GPIO pour actionner l'étage transistor des deux relais et lire des retours du circuit RF. Un ESP32-C3 Super Mini pourrait remplacer ces fonctions via des GPIO reconfigurés.

**Hypothèse B — MCU principal :** U1 est responsable des relais, des boutons LEARN et de la radio ; le CB2S échange seulement un protocole (UART ou autre) avec U1. Un ESP32-C3 doit alors implémenter **ce protocole**, et non piloter les transistors au hasard. C'est plausible au vu des pistes et de U1, **pas confirmé**.

### Relevé électrique hors tension recommandé

1. **Débrancher toute source** : borne IN AC-DC, USB/FT232, alimentation de laboratoire ; vérifier la décharge des condensateurs.
2. Au multimètre en **continuité**, relever pour chaque pad **CB2S 3V3/GND/P6/P7/P8/P10(RX1)/P11(TX1)/P23/P24/P26/CEN** la destination sur la carte mère (entrée U1, résistance série, transistor, WF-LED, etc.). **Ne pas faire de test au hasard sur une carte sous tension.**
3. Repérer quelles pistes passent par une **résistance** : un ohmmètre ne sonnera pas forcément ; la photo et la mesure de résistance complètent le test.
4. Tracer le signal de commande des deux relais jusqu'aux transistors, **sans confondre l'alimentation bobine 5 V et l'entrée de commande de leur transistor**. On cherche les entrées logiques, pas les contacts NO/COM.
5. Établir la correspondance avec le connecteur série (RX1/TX1) : continuité vers U1 ? Si oui, étudier le protocole avant tout échange de module.
6. **Ne pas alimenter la carte de relais depuis l'ESP32-C3 Super Mini** tant que ses rails 3,3 V/5 V, le courant nécessaire et les références de masse ne sont pas identifiés. Programmer et tester le C3 **isolé via USB-C**, puis déterminer l'alimentation finale sans simultanéité de sources non protégée.
7. Si l'objectif est un remplacement, **retirer ou isoler complètement le CB2S**, plutôt que couper seulement GND en laissant les autres GPIO actifs. Préserver la possibilité de remonter le module ; aucun dump Tuya disponible.

### Décision et limites

- Le choix utilisateur est **ESP32-C3 Super Mini USB-C** ; **ne pas proposer par défaut un ESP8685-WROOM-03**.
- Les **GPIO3/GPIO4** mentionnés comme exemples sur la Super Mini ne sont **pas** des correspondances de carte validées : **aucune connexion de ces GPIO aux relais n'est autorisée sans mesure**.
- **Pas de mise sous tension de la motorisation du garage** avant vérification de l'état de chaque sortie au boot/reset/OTA et de la durée d'impulsion.
- **Étape suivante concrète :** relever à l'ohmmètre **où aboutissent RX1/P10 et TX1/P11**, puis au moins les autres pistes CB2S vers U1/transistors. Cela distinguera GPIO directs et communication série.


## 16/11/2026 — essai pratique : récepteur RF autonome sans CB2S

**Observation rapportée par l'utilisateur :** lors d'un essai séparé de la carte de réception `KG21WF+RF V2.0`, la télécommande radio reste utilisable **sans le module Wi-Fi CB2S présent**. C'est un résultat matériel qui prime sur les hypothèses précédentes : **la réception radio et au moins la fonction commandée restent opérationnelles sans firmware Tuya**. Le nombre exact de relais effectivement testés, leur état temporisé ou bistable et l'isolement électrique des sorties ne sont pas encore détaillés ; ne pas les marquer « tous deux validés » sans vérification.

### Conséquence sur l'architecture envisagée

**Objectif clarifié :** conserver intacte l'utilisation des télécommandes RF et ajouter avec **ESP32-C3 Super Mini + ESPHome** un second moyen de déclencher à distance le même événement (comme une impulsion de commande), sans réimplémenter le protocole radio ni dépendre du cloud Tuya.

Deux architectures possibles, classées pour l'essai :
1. **Solution indépendante et peu intrusive :** ajouter une sortie de contact sec, via un **petit relais auxiliaire commandé par l'ESP32-C3** (ou deux si CH1/CH2 sont nécessaires), **en parallèle sur le contact NO/COM correspondant à la sortie d'origine**, seulement si la fonction électromécanique et le branchement aval sont validés. Le récepteur RF et son circuit de relais restent inchangés ; les deux contacts produisent la même fermeture momentanée. Il faut fournir une alimentation adaptée à l'étage relais (une bobine 5 V ne doit jamais être branchée directement à une sortie GPIO 3,3 V).
2. **Solution intégrée plus compacte, mais à cartographier :** identifier sur le PCB le signal logique que le décodage RF envoie à l'étage de commande, puis utiliser une interface compatible (optocoupleur/transistor correctement dimensionné et référencé) en **commande parallèle câblée**. Ne jamais relier directement au hasard un GPIO ESP32 aux sorties du contrôleur U1 : polarités, tensions et éventuelles sorties push-pull inconnues.

La **solution 1** ne nécessite pas de connaître les GPIO historiques du CB2S, et constitue le prototype le plus isolé et réversible. La **solution 2** peut devenir préférable si le projet privilégie l'encombrement minimal après cartographie.

**Point de vigilance important :** `LEARN1` et `LEARN2` semblent être les boutons d'association RF, **pas** des commandes d'ouverture/fermeture. Ne pas les court-circuiter pour simuler la télécommande avant caractérisation de leur comportement.

**Validation de sécurité :**
- Vérifier d'abord les **deux sorties séparément** avec le récepteur RF seul, multimètre en continuité ou charge de test adaptée, installation moteur **déconnectée** ; mesurer durée et type d'action (impulsion, maintien, bascule).
- Déterminer ce que contrôlent réellement **CH1 et CH2** sur le dispositif : ouvrir/fermer, impulsions indépendantes ou deux canaux génériques. Une simple impulsion de portail peut agir en séquence « ouvrir → stopper → fermer » selon l'automate en aval ; ne pas promettre une commande « ouvrir » ou « fermer » dédiée sans retour d'état.
- Le programme ESPHome doit assurer une **impulsion courte bornée et un état inactif au démarrage**, sans déclenchement intempestif pendant Wi-Fi, OTA, coupure/reprise alimentation.
- Ne pas faire de tests sous tension sur une carte ouverte reliée au moteur ou au secteur, et **ne pas faire circuler de tension externe depuis la carte moteur vers un GPIO ESP32**.

**Statut** : autonomie RF *observée par l'utilisateur* ; interface Wi-Fi ESP32-C3 **non raccordée**, durée des impulsions et canaux réels à mesurer ; aucun dump Tuya obtenu. Ce constat réduit l'intérêt de poursuivre les échecs de `ltchiptool`, mais ne justifie pas de relier les sorties de commande sans schéma.


## Correction de stratégie — ESP32-C3 en remplacement logique direct du CB2S (16/11/2026)

**Argument utilisateur confirmé :** le CB2S et l'ESP32-C3 Super Mini fonctionnent tous deux avec des **signaux logiques 3,3 V**. Puisque la carte réceptrice RF sait déjà faire fonctionner les relais **sans CB2S**, l'objectif prioritaire est de **remplacer le module Wi-Fi par une interface GPIO/UART équivalente directement sur les signaux logiques déjà utilisés**, en conservant la carte KG21WF+RF V2.0, son MCU et ses étages de puissance.

**Révision du conseil précédent :** la proposition de petit relais à contact sec ajouté en parallèle de CH1/CH2 est une solution de contournement facultative, **pas une nécessité technique**, et ne correspond pas au souhait de l'utilisateur de remplacer le CB2S. Ne pas réintroduire cette proposition comme solution par défaut.

### Précautions électriques, malgré le même 3,3 V

- **Même tension logique ≠ brochage identique** ; il faut mapper chaque pad CB2S vers la **fonction réelle** sur la carte mère, puis choisir des GPIO ESP32-C3 disponibles.
- **Même tension ≠ même capacité en courant** : Espressif donne des capacités de commande distinctes selon les GPIO et leur mode de sortie. Une entrée de logique CMOS à haute impédance est normalement compatible, mais vérifier la charge, la polarité et les éventuelles sorties en conflit.
- **Même logique ≠ même protocole** : si U1 dialogue avec le CB2S par UART (Tuya MCU), il faut reproduire les échanges (ou déterminer une entrée logique physique utilisable), pas seulement inverser des niveaux.
- **Au redémarrage**, GPIO ESP32-C3 initialement en entrée/haute impédance ou avec résistances de rappel et broches de strapping particulières (GPIO2/8/9). Les états sûrs du CH1/CH2 doivent être assurés dans **le circuit et la configuration**, y compris avant l'initialisation ESPHome.
- **Alimentation** : la carte Super Mini complète peut présenter un appel de courant Wi-Fi différent du module CB2S. Vérifier la régulation 3,3 V disponible, les masses et l'absence d'alimentation USB/externe concurrente. Ne jamais raccorder directement un GPIO 3,3 V à la bobine 5 V du relais.
- Le CB2S actuellement partiellement isolé de sa carte mère ne doit pas rester une source de conflit sur les GPIO : déposer/isoler réellement les signaux concernés avant liaison avec le C3.

### Séquence de diagnostic désormais prioritaire

1. **Hors tension**, relever quelles broches du CB2S aboutissent à U1 (notamment P10/RX1 et P11/TX1), puis aux entrées de commande des relais, transistors et LED.
2. Avec récepteur RF autonome et sans moteur raccordé, caractériser CH1 et CH2 (durée d'impulsion, état de repos, déclenchement à la télécommande).
3. Décider **GPIO logique direct ou protocole série** selon ces preuves, et seulement après réaliser une table **fonction carte mère → GPIO ESP32-C3 Super Mini**.
4. Tester le C3 **seul sur USB-C** avec un firmware neutre, sorties non actives ; intégrer ensuite les commandes à la carte et vérifier l'absence de déclenchement au boot/Wi-Fi/OTA.

**Références** : [Tuya CB2S datasheet, fonction des broches](https://developer.tuya.com/en/docs/iot/cb2s-module-datasheet?id=Kafgfsa2aaypq) et [Espressif ESP32-C3 datasheet, capacités et états GPIO au reset](https://documentation.espressif.com/esp32-c3_datasheet_en.html).


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
