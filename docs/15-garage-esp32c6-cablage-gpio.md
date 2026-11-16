# HFY-KG21WF+RF — passage ESP32-C6 et réservation des GPIO (16/11/2026)

## Changement de carte et état constaté

L'utilisateur abandonne **ESP32-C3 Super Mini**, qui n'arrivait pas à s'associer au SSID Wi-Fi `IoT`, et retient un **ESP32-C6** qui, lui, **se connecte correctement** avec une configuration dérivée de la V1. **La référence exacte de la carte porteuse C6 n'a pas encore été photographiée**. Les correspondances de pastilles ci-dessous supposent une variante **ESP32-C6 Super Mini** de type MakerGO ou compatible, à confirmer par les marquages physiques.

Nouveau [YAML de référence C6 V1](../firmware/hfy-kg21wf-rf/garage-wifi-c6-v1.yaml) (aucune sortie GPIO activée). Il utilise `board: esp32-c6-devkitc-1`, `variant: ESP32C6`, `framework: esp-idf`, cohérents avec ESPHome. **Il n'a pas été compilé/flashé sur l'exemplaire de l'utilisateur dans cette conversation** : ne pas confondre la connexion de son firmware actuel avec une validation de notre fichier GitHub.

**Ne pas re-flasher sans raison une carte qui fonctionne** : conserver le nom ESPHome, les clés API et les secrets déjà utilisés. Le YAML de référence emploie un nouveau nom et des clés `garage_c6_*` et doit être adapté s'il s'agit d'une migration d'un dispositif déjà intégré à Home Assistant.

## GPIO ESP32-C6 — risques de boot, USB et debug

D'après le [datasheet officiel ESP32-C6](https://documentation.espressif.com/esp32-c6_datasheet_en.html) et [Espressif Schematic Checklist](https://docs.espressif.com/projects/esp-hardware-design-guidelines/en/latest/esp32c6/schematic-checklist.html) :

- **GPIO4, GPIO5, GPIO8, GPIO9, GPIO15** : **strapping**, déconseillés pour la liaison avec le contrôleur de portail (un niveau imposé lors de la mise sous tension peut modifier les réglages du boot).
- **GPIO12, GPIO13** : USB Serial/JTAG D-/D+, garder pour flash, programmation et logs.
- **GPIO16, GPIO17** : UART0 de diagnostic, réserver.
- **GPIO6/GPIO7** : JTAG, possibles en GPIO après configuration, mais moins prioritaires.
- **GPIO0, GPIO1, GPIO2, GPIO3, GPIO14** : candidats privilégiés car pas de strapping et non affectés aux circuits USB/UART0, **à condition qu'ils soient réellement exposés sur la carte C6 de l'utilisateur** et libres de circuits auxiliaires ; **cela ne garantit pas l'absence de transitoire externe ni un relais inactif au boot**.
- Sur plusieurs **C6 Super Mini**, GPIO15 est relié à une LED, GPIO8 à une LED RGB, GPIO9 au bouton BOOT. Cela peut varier selon la révision/constructeur : contrôler la sérigraphie.

## Table de réservation : ancien CB2S → futur ESP32-C6

| Ancien CB2S / zone | GPIO C6 envisagé | Usage attendu | État |
| --- | --- | --- | --- |
| **P6** | **GPIO0** | Premier signal vers HC89S003A | Hypothèse, mode/sens non identifié |
| **P7** | **GPIO1** | Deuxième signal vers HC89S003A | Hypothèse, mode/sens non identifié |
| **P8** | **GPIO2** | Troisième signal vers HC89S003A | Hypothèse, mode/sens non identifié |
| **P24** | **GPIO3** | Réseau de diodes D5/D6, rôle inconnu | À cartographier avant raccordement |
| **P26** | **GPIO14** | `WF-LED`, polarité et résistance série à déterminer | À cartographier avant raccordement |
| **P10/RX1** | Non prévu | À réserver si dialogue série avéré | Non câbler par défaut |
| **P11/TX1, P23, CEN** | Non prévu | Aucune fonction de commande établie | Ne pas reproduire ces lignes par défaut |

**Cette table réserve des GPIO dans le projet, mais ne prouve pas les fonctions de P6/P7/P8**. On sait seulement (d'après observation à la loupe) qu'ils convergent vers U1. Les diodes D5/D6 pourraient être un réseau de commande, mais ce n'est pas établi. `WF-LED` peut indiquer une commande Wi-Fi, pas nécessairement une association Wi-Fi.

### Ne pas souder les GPIO de commande à ce stade

- Établir d'abord **tension U1 (possiblement 5 V)**, tensions réelles des lignes, topologie **entrée/sortie**, polarité active, présence de résistances et comportement en boot.
- **Même tension de sortie 3,3 V** du CB2S et du C6 ne prouve **ni mêmes courants disponibles, ni compatibilité d'une entrée U1 5 V, ni absence de conflit électrique**. Ne pas relier deux sorties push-pull.
- Le C6 doit être **préflashé/testé seul** puis raccordé de manière sûre : retirer réellement le CB2S de la carte (une simple coupure de masse ne suffit pas), alimentation régulée 3,3 V dimensionnée (le C6 peut tirer des pics au Wi-Fi), GND commun ; **ne pas brancher simultanément USB et alimentation externe** sans gestion adéquate.
- Avant toute commande de portail : essais sans moteur, état inactif au power-on, brownout/reset, OTA et perte de Wi-Fi, et vérification de la durée d'impulsion.
- Pour l'instant, **V1 n'active aucun GPIO** ; cela ne constitue pas une garantie matérielle si une entrée U1 est laissée flottante.

## Étapes suivantes

1. Vérifier que la carte est bien **ESP32-C6 Super Mini**, et localiser physiquement IO0/1/2/3/14 (pas uniquement selon la photo d'un autre fabricant).
2. Mesurer **hors tension** l'interface de P6, P7, P8, P24, P26 vers le HC89S003A et la carte mère ; confirmer résistances/diodes.
3. À la mise en service avec récepteur RF seul et charge moteur déconnectée, relever **les niveaux** et les effets des commandes RF avec instrument adapté et méthodes isolées, sans injecter de signaux inconnus.
4. Créer une **V2** qui utilise uniquement les GPIO démontrés, avec polarités et états de repos validés.

**Sources :** [Espressif datasheet ESP32-C6](https://documentation.espressif.com/esp32-c6_datasheet_en.html) ; [Espressif strapping/boot](https://docs.espressif.com/projects/esp-hardware-design-guidelines/en/latest/esp32c6/schematic-checklist.html) ; [ESPHome ESP32 Platform](https://esphome.io/components/esp32/) ; [C6 Super Mini retours de brochage MakerGO](https://github.com/mboehmerm/Three-IPS-Displays-with-ST7789-170x320-240x280-240x320/blob/main/ESP32_C6_SuperMini/README.md).
