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


## Essais recommandés, en conservant les relais et le garage hors charge

1. Conserver **GPIO3 en entrée haute impédance**, sans pull-up/down, dans la V2.
2. Vérifier la tension sur P6/GPIO3 (à l'état haut et durant la commande) **par rapport au GND commun**, idéalement oscilloscope ou analyseur logique 3,3 V. Tout signal supérieur à 3,3 V met l'ESP en danger.
3. Avec multimètre en mode continuité **hors tension**, cartographier le circuit de P6 jusqu'à la patte U1 (résistances, éventuels transistors) afin de déterminer si U1 peut conduire cette ligne en sortie.
4. Pour comprendre le canal 2, tester **séparément la télécommande du second canal** et relever les événements de toutes les lignes, surtout P7 et P8.
5. Si le but est d'ajouter la commande Wi-Fi : **ne pas provoquer de niveau bas artificiel sur P6** avant preuve qu'il s'agit d'une entrée et choix d'une interface de sécurité adaptée. Déterminer une **entrée de commande** réelle ou un protocole U1 puis seulement concevoir une V3 de contrôle.

**Statut : P6 = indicateur actif bas lié au relais confirmé par tests utilisateur, direction entrée/sortie inconnue. Interdiction de sortie GPIO3 jusqu'à vérification.**
