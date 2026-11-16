# Contrôleur HFY-KG21WF+RF V2.0 — identification de U1 (16 novembre 2026)

## Résultat de la recherche

**Composant candidat très probable : Holychip HC89S003A / HC89S003AF4, boîtier TSSOP20.** Le marquage lu par l'utilisateur était `HC895003A` suivi de `2510T` : le caractère `5` peut en réalité être un `S`. Sur la photographie rapprochée de U1, on compte **10 pattes par côté, donc 20**, compatibles avec la fiche officielle. **L'identification n'est pas totalement garantie** tant qu'on n'a pas confirmé le marquage ou les liaisons des broches d'alimentation/test.

La mention historique « U1 à 28 broches » dans notre analyse était **une erreur de comptage**.

**Fiche constructeur** : [Holychip HC89S003A/001A — documentation PDF et pinout TSSOP20](https://taoic.oss-cn-hangzhou.aliyuncs.com/9007/product/zanchip888_1629191116000.pdf), pages imprimées **6, 9, 10** (capacités, configuration et description des broches). Autre [fiche anglaise de la même famille](https://datasheet4u.com/pdf-down/H/C/8/HC89S003AF4-Holychip.pdf).

### Ce que permet le HC89S003A

- Microcontrôleur **8051 8 bits**, mémoire **16 Ko Flash + 128 octets EEPROM**, jusqu'à **18 GPIO**, alimentation **2,0 à 5,5 V**.
- **2 UART, SPI, bus I²C**, timers, PWM, ADC, ainsi que programmation/débogage deux fils / JTAG.
- **PTM : les fonctions de communication peuvent être mappées sur différentes broches par le firmware**, ce qui interdit de déterminer le protocole exact à partir du modèle de puce seul.
- L'alimentation **réelle de U1** sur la KG21WF+RF n'est pas connue : les relais sont équipés de bobines 5 V et le CB2S est 3,3 V, mais **U1 pourrait fonctionner à 3,3 V ou 5 V**. Ne jamais connecter les GPIO de la Super Mini avant de mesurer VDD de U1 et les signaux.

### Brochage TSSOP20 utile au diagnostic

La numérotation du boîtier commence **patte 1 à l'angle voisin du repère rond / encoche**, descend 1 → 10 sur ce côté puis remonte 11 → 20 sur le côté opposé.

| Patte U1 | Fonction HC89S003A | Sur la carte KG21WF+RF V2.0 |
| --- | --- | --- |
| **4** | **P0.3 / SCK** (horloge interface 2 fils) | Candidate pour la pastille **CLK** |
| **7** | **GND** | Candidate pour la pastille **GND** |
| **9** | **VDD** | Candidate pour la pastille **VCC** |
| **12** | **P2.7 / RST** | Candidate pour la pastille **RST** |
| **18** | **P2.1 / SDA** (données interface 2 fils) | Candidate pour la pastille **DAT** |
| 13–16 | TMS/TCK/TDO/TDI et GPIO | Débogage JTAG possible selon configuration, sans preuve de liaison ici |

Ces rapprochements sont **des hypothèses à valider au multimètre en continuité**, pas des connexions déjà mesurées. **Les pastilles DAT/CLK/RST sont vraisemblablement l'interface de programmation du microcontrôleur U1, et non l'UART du CB2S.** Si les cinq correspondances sont retrouvées, la famille identifiée sera beaucoup mieux étayée.

### Observations CB2S à garder distinctes

| Broche CB2S | Relevé utilisateur | Statut |
| --- | --- | --- |
| P6 | Piste vers zone U1 | Non attribuée à un relais / non mesurée |
| P7 | Piste vers zone U1 | Non attribuée |
| P8 | Piste vers zone U1 | Non attribuée |
| P24 | Vers la zone des diodes **D5 et D6** | Fonction des diodes **inconnue** ; possible routage logique, pas une commande validée |
| P26 | Vers **WF-LED** | Voyant lié aux commandes Wi-Fi probable ; ne pas présumer que la LED indique l'état du réseau plutôt que la réception d'une commande |
| P11/TX1, P23, CEN | Aucun usage apparent à la loupe | **À confirmer** ; « pas de piste vue » n'est pas « pas de connexion » |

La **télécommande RF a été observée fonctionnelle sans le CB2S**. Cela soutient fortement l'hypothèse que **U1 exécute déjà la logique RF et relais**. Le CB2S servirait à présenter des ordres Wi-Fi à U1. Ce sont peut-être des entrées GPIO impulsionnelles, mais ni le firmware HC89S003A ni le protocole de l'appareil ne sont connus : **ne pas confondre capacités du MCU et protocole effectivement utilisé**.

### Le premier test recommandé

**Carte intégralement hors tension, borne AC-DC, motorisation et USB déconnectés, condensateurs déchargés :** en mode continuité, vérifier d'abord les associations suivantes sur la rangée de cinq pastilles :

- `GND` → patte **7** U1 ;
- `VCC` → patte **9** U1 ;
- `RST` → patte **12** U1 ;
- `DAT` → patte **18** U1 ;
- `CLK` → patte **4** U1.

Une liaison au travers de résistance peut ne pas biper ; relever la résistance mesurée si c'est le cas. **Ne pas brancher de FT232 sur DAT/CLK** : c'est potentiellement l'interface de programmation du HC89S003A, pas une liaison UART. **Ne pas lire ni reprogrammer U1** : l'objectif est de conserver intacte sa fonction RF.

Ensuite, poursuivre les continuités `CB2S P6/P7/P8/P24 → résistances/diodes → pattes numérotées de U1`, pour cartographier une véritable interface **ESP32-C3 Super Mini → carte d'origine**.

**Aucune commande d'ouverture/fermeture ne doit être générée avant d'avoir identifié et validé les signaux, niveaux et comportement au démarrage du C3.**
