# Diagnostic : ESP32-C6 soudé, 3 V mesurés mais pas de démarrage (16/11/2026)

## Nouvel état matériel rapporté

L'utilisateur a soudé l'ESP32-C6 Super Mini sur la carte **KG21WF+RF V2.0** en reprenant six signaux :

| CB2S d'origine | C6 |
| --- | --- |
| **P10** | **GPIO18** |
| **P24** | **GPIO0** |
| **P26** | **GPIO19** |
| **P6** | **GPIO3** |
| **P7** | **GPIO2** |
| **P8** | **GPIO1** |

L'utilisateur mesure environ **3 V arrivant sur l'ESP32-C6**, mais ne voit **pas de LED clignotante ni de démarrage évident**. Le YAML V1.1 n'affecte **aucune** de ces six broches à une commande : elles ne sont pas configurées en sortie ; cela ne garantit pas pour autant une isolation électrique des signaux.

## Diagnostic électrique prioritaire

1. **Tension alimentation insuffisante ou chute en charge :** ESP32-C6 officiellement alimenté en **3,0–3,6 V**, 3,3 V typique, puissance recommandée adaptée à un courant disponible d'au moins **0,5 A** (datasheet). Une mesure DC de **3,0 V** est à la limite basse et peut cacher un creux à l'initialisation Wi-Fi. L'ancien régulateur 3,3 V destiné au CB2S peut ne pas fournir assez de courant à la carte C6 complète.
2. **Masse coupée lors des essais BK7231N précédents :** l'utilisateur avait **isolé/coupé une liaison GND du module CB2S** pour éviter l'alimentation parasite de la carte mère. Vérifier si la liaison GND entre carte mère, son régulateur et ESP32-C6 est **réellement rétablie**. Une tension lue depuis un point inadéquat peut être trompeuse et les autres fils peuvent rétroalimenter via diodes de protection.
3. **Conflit ou injection de tension sur les six lignes :** ces GPIO ne sont pas des **strapping pins** du C6, donc ne bloquent normalement pas le choix du mode de boot. Mais leur raccordement à une sortie du MCU U1 potentiellement alimenté en 5 V ou à des réseaux de diodes peut imposer une tension inadmissible, court-circuiter des sorties ou provoquer une rétroalimentation.
4. **Broche d'alimentation mal choisie :** s'assurer que le +3,3 V arrive bien sur **3V3** et non **5V** de la carte Super Mini, et que **GND est commun**. La tension mesurée **entre 3V3 et GND du module**, sous charge, est le premier indicateur pertinent.
5. **LED non probante :** la LED de présence d'alimentation n'est pas toujours une LED clignotante ; la LED utilisateur peut nécessiter une déclaration dans ESPHome. Une absence de clignotement ne démontre pas à elle seule que l'ESP est arrêté ; contrôler logs et présence réseau après alimentation correcte.

## Procédure de séparation des causes — carte de motorisation déconnectée

- **Avant toute déconnexion :** couper et isoler **toutes** les alimentations, moteur, USB, secteur éventuel. Vérifier décharge des condensateurs.
- **Test A :** débrancher **les six liaisons GPIO** de la Super Mini (ou les isoler réellement), laisser uniquement **3V3 et GND** depuis la carte mère, si la basse tension et la polarité sont établies. Mesurer **3V3 à GND directement sur la C6 au moment du démarrage** et rechercher le SSID / IP / logs. Ne rien alimenter si le montage est raccordé à un secteur non isolé.
- **Test B :** si le C6 ne démarre toujours pas, **retirer aussi les fils 3V3 et GND de la carte mère**, et alimenter la C6 **seule en USB-C** comme lors de sa connexion Wi-Fi précédente. **Jamais d'USB-C simultanément avec le rail 3V3 externe relié sans dispositif anti-retour vérifié.**
  - C6 fonctionne seul en USB mais pas avec 3V3 carte mère : alimentation/câblage/masse à corriger.
  - C6 fonctionne sur 3V3 carte mère sans GPIO, mais échoue quand un fil est reconnecté : interaction électrique sur le fil concerné, **ne pas continuer les essais de commande**, mesurer tension/sens.
  - C6 ne fonctionne plus même isolé en USB : court-circuit résiduel, soudure pontée, composant endommagé ou firmware ; inspection hors tension puis diagnostic USB. Ne pas conclure sans log.
- **Avant de reconnecter des GPIO :** mesurer les niveaux des fils **côté U1 avec un instrument approprié et une alimentation basse tension isolée**, confirmer **pas de 5 V** sur les entrées C6 et que les signaux ne sont pas activement conduits par les deux côtés ; s'assurer que les sorties CH1/CH2 n'ont aucune charge mécanique raccordée.
- Après identification, reconnecter **un seul signal à la fois**, vérifier boot puis garder la logique de portail inerte au reset, brownout et OTA.

## Priorité et sources

**Ne pas changer le YAML immédiatement.** Le firmware V1.1 ne déclare aucun GPIO physique et le C6 fonctionnait en Wi-Fi avant soudure. Priorité à l'alimentation/masse/soudure et à l'isolation des six signaux.

- [Fiche ESP32-C6 : 3,0 V min, 3,3 V typ, 3,6 V max, alimentation 0,5 A](https://documentation.espressif.com/esp32-c6_datasheet_en.html)
- [Espressif : configuration strapping GPIO4/5/8/9/15](https://docs.espressif.com/projects/esp-hardware-design-guidelines/en/latest/esp32c6/schematic-checklist.html)
- [Schéma ESP32-C6 Super Mini variable selon fabricant](https://docs.espressif.com/projects/esp-dev-kits/en/latest/esp32c6/esp32-c6-devkitc-1/user_guide.html)

**État : diagnostic à faire, aucun nouveau relevé demandé dans cette fiche n'est encore confirmé.**
