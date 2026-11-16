# KG21WF+RF V2.0 — impulsions sur P6 pendant l'usage de la télécommande RF

**Date : 16 novembre 2026.** L'utilisateur signale que lorsqu'il déclenche le récepteur avec la télécommande RF, **P6 se désactive pendant le claquement du relais, puis se réactive**. Cette observation est corroborée par l'export historique Home Assistant `activity (1).csv`, analysé localement **sans publier de données privées**.

## Matériel et mapping actuel

- Contrôleur autonome RF/relais : U1 probable Holychip HC89S003A.
- Ancien CB2S retiré/isolé ; ESP32-C6 Super Mini relié par :
  - P10 → C6 GPIO18
  - P24 → C6 GPIO0
  - P26 → C6 GPIO19
  - **P6 → C6 GPIO3**
  - P7 → C6 GPIO2
  - P8 → C6 GPIO1
- C6 opérationnel après alimentation de sa broche **5V** via un rail 5 V, avec régulation 3,3 V sur sa carte à confirmer. Wi-Fi IoT et Home Assistant actifs.
- Logiciel [V2 d'observation](../firmware/hfy-kg21wf-rf/garage-wifi-c6-v2-observation.yaml) : six GPIO **INPUT**, aucune commande.

## Observations CSV — faits

L'export comporte 52 événements au total, dont **14 transitions OFF→ON** de P6 durant les essais, avec **durées OFF calculées de 102 à 116 ms, moyenne environ 112–113 ms** (les horodatages sont ceux d'événements Home Assistant, non une mesure d'oscilloscope). P6 passe de **ON → OFF (~0,11 seconde) → ON**, à plusieurs reprises et de façon répétable. Lors de ce relevé, les autres capteurs P7, P8, P10, P24, P26 n'ont qu'un état initial, sans transition rapportée. L'export comporte aussi des périodes d'indisponibilité/reconnexion Wi-Fi et API : ne pas confondre ces événements avec des commandes.

## Interprétation : important de vérifier le sens du signal

- **P6 est corrélé au déclenchement RF du relais**, et **actif bas** dans l'observation. Cela ne prouve pas qu'il **commande** le relais.
- Comme la carte mère exécute les ordres RF et commute le relais **sans l'ancien CB2S**, la ligne **pourrait être une sortie de U1 destinée à notifier le CB2S**. Dans ce cas, passer l'ESP32-C6 GPIO3 en sortie risquerait une collision de sorties.
- Autre possibilité : ligne de statut de relais, signal multiplexé/retour, ou circuit câblé partagé. Identifier la topologie électrique avant simulation.
- La durée d'environ 112 ms est issue de l'historique HA et inclut le polling/horodatage logiciel ; ce n'est pas une mesure électronique certifiée à la milliseconde.

## Complément de test utilisateur : canal Learn 2 et LED Wi-Fi

L'utilisateur a appairé une **seconde télécommande sur Learn 2** et a constaté que **P8 (ESP32-C6 GPIO1) présente le même comportement d'impulsion active bas que P6 (GPIO3)** pendant le relais du deuxième canal.

**Cartographie fonctionnelle mieux étayée :**
- **P6 → GPIO3** : impulsion active bas corrélée à l'actionnement **RF canal 1**, 102–116 ms sur historique HA du premier essai ;
- **P8 → GPIO1** : impulsion active bas corrélée à l'actionnement **RF canal 2**, durée non encore mesurée précisément ;
- **P7 → GPIO2** et **P10 → GPIO18** : hypothèse de l'utilisateur selon laquelle ce sont les **deux entrées de commande Wi-Fi** du Holychip ; **non confirmé**. Ces broches restent en entrées dans le firmware de diagnostic, car forcer P10 ou P7 en sortie pourrait créer un conflit avec une sortie U1 ;
- **P24 → GPIO0** : rôle encore inconnu ;
- **P26 → GPIO19** : utilisateur confirme le routage vers la LED `WF-LED` et demande un flash visible lorsque le C6 est connecté au Wi-Fi.

**Firmware de test créé :** [V3 — LED Wi-Fi P26 sur GPIO19](../firmware/hfy-kg21wf-rf/garage-wifi-c6-v3-led-wifi.yaml). À chaque intervalle de **2 s**, si `wifi.connected`, GPIO19 active brièvement la LED pendant **150 ms**, puis l'éteint ; sinon reste inactif. Le code suppose une LED **active LOW** (`inverted: true`) ; **à vérifier avant activation matérielle**. Les cinq autres broches restent en **entrées haute impédance** pour lecture passive (pas de `switch:` relais).

**Précondition impérative :** vérifier par mesure/inspection que le réseau P26 de la carte est une LED dotée d'une **résistance série** et n'est pas une sortie de l'Holychip U1 ou une autre charge ; **même un GPIO non-strapping peut endommager un circuit s'il attaque une autre sortie**. Ne pas raccorder la motorisation aux relais pour ces essais. Ce firmware est **archivé sans compilation ni validation sur le matériel**.


## Correction : essais déjà réalisés et résultats validés

**Ne pas répéter l'essai LED.** D'après les essais remontés par l'utilisateur :
- Le bouton **TEST WF-LED seule** sur **P26/GPIO19** donne **un flash bleu bref puis extinction** : fonctionnement validé.
- Les essais de commandes **P7/GPIO2 et P10/GPIO18** avec **impulsions de 100 ms** font flasher la LED bleue mais **aucun des deux relais ne claque** : ces broches ne sont **pas validées comme commandes directes**, et l'impulsion LED est déclenchée par le programme, non une preuve de relais.
- Par télécommande, le relais 1 fait réagir **P6/GPIO3**, le relais 2 **P8/GPIO1**, avec impulsion active bas d'environ 0,1 s ; les relais réagissent bien aux ordres RF.

**Prochaine question à résoudre** : P6 et P8 sont-ils des **sorties d'état du Holychip**, des entrées de commande, ou des lignes communes bidirectionnelles ? Ne **pas** convertir GPIO3/GPIO1 en sorties push-pull ; risque d'opposition électrique avec U1.

### Essai ciblé sur P6, avant toute modification YAML
- Moteur/automatisme entièrement déconnecté, carte sur alimentation basse tension **isolée** et correctement limitée en courant ; confirmer la tension de repos de **P6 à GND** au multimètre : elle doit être dans la plage 0–3,3 V compatible C6.
- Si niveau de repos stable et compatible, **tester brièvement un rappel faible à la masse à travers une résistance série de 4,7 kΩ** directement entre **P6 côté circuit** et GND, tout en observant relais 1 et LED RF. Le test **n'est pas** un court-circuit franc ; si P6 est une sortie forte, la résistance limite le courant à ~0,7 mA à 3,3 V, mais le résultat peut ne pas franchir le seuil logique, donc **un échec est non concluant**.
- Mesurer la tension de P6 pendant ce rappel. Une résistance 4,7 kΩ ne protège pas contre une erreur de masse, l'application secteur, ou un circuit exposé à du 5 V : **abandonner si les tensions ne sont pas connues et isolées**.
- **Ne pas recâbler la sortie C6** pendant ce test ; GPIO3 reste configuré **INPUT** dans le YAML pour observer l'état.
- Selon résultat, envisager un essai symétrique P8, puis seulement créer V5 de commandes conditionnées en open-drain, en sécurisant les états au démarrage.


## V5 demandée — impulsion brève sur P8, sans présumer de sa fonction

L'utilisateur demande explicitement de générer une impulsion sur **P8 du CB2S**, aujourd'hui raccordé à **GPIO1 de l'ESP32-C6**. Bien que **P8 réagisse lui-même à la commande du canal RF2**, sa direction reste inconnue : **une ligne qui réagit comme sortie de statut n'est pas forcément une entrée de commande**.

Nouvelle version expérimentale : [garage-wifi-c6-v5-test-p8.yaml](../firmware/hfy-kg21wf-rf/garage-wifi-c6-v5-test-p8.yaml), **non compilée ni testée sur le matériel**.

- **GPIO1/P8** configuré en sortie **open-drain, inverted**, repos haute impédance, impulsion **LOW 80 ms** sur un bouton de test.
- Bouton `TEST P8 - impulsion 80 ms` disponible dans HA **uniquement fonctionnel si armement `AUTORISER test P8 (banc uniquement)` activé** ; armement OFF par défaut et au redémarrage.
- Le `binary_sensor` **P8** est **retiré** : ESPHome ne peut pas utiliser simultanément GPIO1 comme entrée `binary_sensor` et sortie `output` ; les autres GPIO P6/P7/P10/P24 restent en lecture.
- P26/GPIO19 conserve la fonction LED avec flash lors du bouton d'essai. Aucun P7 ni P10 n'est piloté en V5.
- **Condition matérielle impérative** : carte motorisation déconnectée, alimentation de banc isolée, vérification P8 dans 0–3,3 V, **intercaler une résistance 4,7 kΩ en série entre le fil GPIO1 et P8 avant essai** (l'utilisateur avait auparavant une liaison directe), masse commune. La résistance limite le courant potentiel d'opposition, ne garantit pas compatibilité logique ni sécurité.
- Pendant boot/OTA, aucun relais ne doit s'actionner. Si P8 ne donne aucune réaction, **ne pas augmenter arbitrairement la durée ou la force de commande** : cela suggère que P8 est peut-être un retour d'état ; reprendre les mesures du circuit.

**Cet essai ne doit être utilisé que pour le diagnostic sur banc** ; la réussite ou l'échec ne remplace pas le relevé du schéma logique ni la vérification d'entrée/sortie du Holychip.


## Essais recommandés, en conservant les relais et le garage hors charge

1. Conserver **GPIO3 en entrée haute impédance**, sans pull-up/down, dans la V2.
2. Vérifier la tension sur P6/GPIO3 (à l'état haut et durant la commande) **par rapport au GND commun**, idéalement oscilloscope ou analyseur logique 3,3 V. Tout signal supérieur à 3,3 V met l'ESP en danger.
3. Avec multimètre en mode continuité **hors tension**, cartographier le circuit de P6 jusqu'à la patte U1 (résistances, éventuels transistors) afin de déterminer si U1 peut conduire cette ligne en sortie.
4. Pour comprendre le canal 2, tester **séparément la télécommande du second canal** et relever les événements de toutes les lignes, surtout P7 et P8.
5. Si le but est d'ajouter la commande Wi-Fi : **ne pas provoquer de niveau bas artificiel sur P6** avant preuve qu'il s'agit d'une entrée et choix d'une interface de sécurité adaptée. Déterminer une **entrée de commande** réelle ou un protocole U1 puis seulement concevoir une V3 de contrôle.

**Statut : P6 = indicateur actif bas lié au relais confirmé par tests utilisateur, direction entrée/sortie inconnue. Interdiction de sortie GPIO3 jusqu'à vérification.**
