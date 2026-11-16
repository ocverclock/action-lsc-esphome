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
