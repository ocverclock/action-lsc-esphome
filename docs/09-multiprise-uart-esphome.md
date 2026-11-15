# Multiprise TV (LSC 5225000500) — préparation UART → ESPHome

**État au 15/11/2026 : code préparé, matériel français encore fermé ou en cours d'ouverture, carte NON confirmée, firmware NON compilé et NON flashé.** Ce document n'autorise aucune mise sous tension d'une carte ouverte.

## Correspondance documentaire

L'appareil de l'utilisateur (« Multiple TV ») a : modèle Tuya **5225000500**, product_id **nflvklbkgbbxegxj**, firmware **Module V1.1.17 / Autre V1.0.5**, **quatre sorties DP1–DP4**, protocole LAN **3.4** lu avec succès (DP1=false, DP2=false, DP3=false, DP4=true au moment du relevé). Son aspect extérieur correspond à la multiprise Action 3202887 publiée, sauf les prises de terre françaises (type E).

Sur la [multiprise Action 3202887 démontée](https://www.elektroda.com/rtvforum/topic4080696.html) :
- **CB3S, BK7231N**, flash **2 MiB** (à confirmer sur l'appareil de l'utilisateur).
- Relais **P14, P6, P7, P8** (ordre logique décrit dans le démontage), **bouton commun P10**, **LED Wi-Fi P23 active LOW**. L'orientation physique, le brochage réel et les polarités **ne sont pas confirmés sur l'exemplaire français**.
- [Autre démontage](https://www.elektroda.com/rtvforum/topic4095176.html) : alimentation potentiellement **non isolée du secteur 230 V** ; vis de sécurité et boîtier difficiles à démonter et à remonter.
- [Carte CB3S / brochage officiel LibreTiny](https://docs.libretiny.eu/boards/cb3s/): UART1 **P10 RX1** (TX du FTDI), **P11 TX1** (RX du FTDI), **CEN** réinitialisation. Attention, P10 sert aussi au bouton physique : le câble UART doit être débranché avant la remise en service.
- [Flash série / sauvegarde ltchiptool](https://docs.libretiny.eu/docs/flashing/tools/ltchiptool/) : sauvegarde complète en lecture avant toute écriture ; les images ESPHome pour l'UART sont typiquement des **.uf2** compilés pour CB3S, pas l'image d'exploit **.ota.ug.bin**.
- [Issue CloudCutter #714](https://github.com/tuya-cloudcutter/tuya-cloudcutter/issues/714) : firmware **1.1.17 / 1.0.5** corrigé contre l'ancien exploit ; OTA UnTuyaOS3 **non démontré**, raison du choix de préparer la voie série.

## Prototype ESPHome

[Code candidat multiprise-tv-candidate.yaml](../firmware/5225000500/multiprise-tv-candidate.yaml) :

- **4 interrupteurs GPIO distincts** : P14, P6, P7, P8, configuration initiale `restore_mode: ALWAYS_OFF`.
- Carte `bk72xx: board: cb3s` (si matériel confirmé), pas `generic-bk7238-tuya`.
- **Interface Web locale**, sans HA ni accès Internet, ressources locales, HTTP Digest protégé par mot de passe ; l'API HA reste facultative.
- Un seul Wi-Fi IoT 2,4 GHz (`wifi_ssid_IoT` / `wifi_password_IoT` déjà utilisés dans les autres projets), **AP secours chiffré**.
- API clé unique et **OTA ESPHome chiffré** (`encryption:`) avec cette même clé ; aucune nécessité d'un mot de passe OTA séparé.
- **Bouton physique P10 désactivé dans le prototype** jusqu'à vérification polarité/anti-rebond et choix des actions « tout allumer/éteindre ». Le bouton Web « Tout eteindre » est disponible.
- Voyant d'état P23 supposé inversé ; vérifier.
- `ALWAYS_OFF` définit un **état logiciel**, pas une preuve d'absence de commutation transitoire ni du bon niveau physique. Un GPIO inversé peut produire l'effet opposé.

Secrets à créer **seulement dans ESPHome /config/esphome/secrets.yaml**, valeurs jamais déposées sur GitHub :

```yaml
multiprise_tv_api_key: "BASE64_ALEATOIRE_32_OCTETS"
multiprise_tv_ap_password: "MOT_DE_PASSE_AP_SECOURS"
multiprise_tv_web_password: "MOT_DE_PASSE_WEB"
# Les noms ci-dessous sont ceux deja utilises sur les prises :
wifi_ssid_IoT: "NOM_DU_WIFI_IOT"
wifi_password_IoT: "MOT_DE_PASSE_DU_WIFI_IOT"
```

Génération locale (ne pas publier le résultat) :

```bash
python3 -c 'import os,base64; print(base64.b64encode(os.urandom(32)).decode())'
openssl rand -hex 16  # mot de passe AP ou Web, distinct par usage
```

**Validation ESPHome à réaliser avant flash :** importation du YAML dans Device Builder, résolution de tous les `!secret`, bouton **Validate**, compilation **Install → Manual download** et génération du `firmware.uf2`. Fichier documenté, **pas encore validé ni compilé** : des adaptations peuvent s'imposer à la version ESPHome utilisée.

## Plan de conversion physique, par intervenant compétent

1. Débrancher **toutes les charges et la prise secteur**. Sur une carte d'alimentation secteur non isolée, une intervention nécessite des compétences électriques : vérifier l'absence de tension résiduelle et l'état de sécurité avant tout contact. Ne jamais connecter l'UART au PC quand la multiprise est reliée au secteur.
2. Photographier les deux faces du PCB et le CB3S, étiquettes et pistes, sans apporter d'alimentation. Comparer la révision et les broches avec le démontage publié ; relever également la tension de commande des relais et le chemin d'alimentation.
3. Vérifier au multimètre les **niveaux logiques de l'adaptateur FT232** (3,3 V) et disposer d'une **alimentation stabilisée 3,3 V dimensionnée** ; la sortie 3,3 V de certains FT232RL peut être insuffisante.
4. Après identification formelle du **CB3S**, préparer la liaison série UART1 **TX croisé avec RX**, GND commun, alimentation 3,3 V dédiée, **aucun 230 V**.
5. Priorité **sauvegarde du firmware d'origine** avec ltchiptool : ouvrir l'option **Read flash** (famille BK7231N, généralement **2 MiB**) ou la commande CLI affichée ci-dessous. Les sauvegardes peuvent contenir des secrets Tuya/Wi-Fi : **ne pas les mettre sur GitHub**.
6. Ne passer à **Write flash** qu'après sauvegarde vérifiée, PCB/polarités contrôlés, YAML validé/compilé et fichier UART `firmware.uf2` vérifié (famille BK7231N).
7. Tester d'abord l'amorçage logique avec une alimentation 3,3 V isolée **sans connexion 230 V**, puis débrancher totalement le programmateur, remonter le boîtier et vérifier sa sûreté avant de tester les sorties avec une charge d'essai appropriée sur secteur, boîtier **fermé**.
8. Vérifier **chaque sortie indépendamment**, l'ordre physique, le bouton, la LED, le redémarrage et les coupures ; dernière étape : **WAN bloqué**, commande locale via interface Web, sans HA et sans Tuya. Ne rebrancher les équipements TV qu'une fois les quatre voies validées.

Exemples d'aide ltchiptool, sans effet destructif :

```bash
ltchiptool --help
ltchiptool list families
ltchiptool flash read --help
ltchiptool flash write --help
```

Pour une sauvegarde **après validation de l'identité du CB3S, du port et de l'alimentation hors secteur** :

```bash
ltchiptool flash read -d /dev/ttyUSB0 bk7231n sauvegarde-multiprise-originale.bin
```

**Ne pas lancer** `flash write` tant que l'appareil utilisateur n'a pas été photographié et que la compilation ESPHome n'est pas vérifiée.

## Conditions d'acceptation

- [ ] PCB CB3S/BK7231N et révision réelle confirmés.
- [ ] Sauvegarde du firmware original obtenue et conservée hors GitHub.
- [ ] GPIO, polarités, ordre des relais et bouton vérifiés.
- [ ] YAML validé et image UF2 compilée pour `cb3s`.
- [ ] Flash série avec aucune liaison au secteur.
- [ ] Boîtier refermé sans compromettre la sécurité mécanique/électrique.
- [ ] Quatre sorties distinctes, chacune OFF au boot, aucune commutation intempestive.
- [ ] Bouton physique commun conforme à la demande.
- [ ] Interface Web locale accessible, OTA et API chiffrées.
- [ ] Fonctionnement sans WAN et sans Home Assistant confirmé.
