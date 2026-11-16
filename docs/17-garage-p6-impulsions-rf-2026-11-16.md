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

## Essais recommandés, en conservant les relais et le garage hors charge

1. Conserver **GPIO3 en entrée haute impédance**, sans pull-up/down, dans la V2.
2. Vérifier la tension sur P6/GPIO3 (à l'état haut et durant la commande) **par rapport au GND commun**, idéalement oscilloscope ou analyseur logique 3,3 V. Tout signal supérieur à 3,3 V met l'ESP en danger.
3. Avec multimètre en mode continuité **hors tension**, cartographier le circuit de P6 jusqu'à la patte U1 (résistances, éventuels transistors) afin de déterminer si U1 peut conduire cette ligne en sortie.
4. Pour comprendre le canal 2, tester **séparément la télécommande du second canal** et relever les événements de toutes les lignes, surtout P7 et P8.
5. Si le but est d'ajouter la commande Wi-Fi : **ne pas provoquer de niveau bas artificiel sur P6** avant preuve qu'il s'agit d'une entrée et choix d'une interface de sécurité adaptée. Déterminer une **entrée de commande** réelle ou un protocole U1 puis seulement concevoir une V3 de contrôle.

**Statut : P6 = indicateur actif bas lié au relais confirmé par tests utilisateur, direction entrée/sortie inconnue. Interdiction de sortie GPIO3 jusqu'à vérification.**
