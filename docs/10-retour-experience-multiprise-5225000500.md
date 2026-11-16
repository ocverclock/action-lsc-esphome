# Retour d'expérience — multiprise LSC 5225000500 / ESPHome (16 novembre 2026)

> **Lecture importante :** observations effectuées sur **un exemplaire français** de la multiprise LSC « Multiple TV » 5225000500. Ne pas extrapoler son brochage à une autre révision sans vérification. Les étapes de mise sous tension d'une carte secteur ouverte sont à proscrire pour toute reproduction ; l'utilisateur a confirmé que le boîtier est refermé pour les essais logiciels suivants.

## 1. Ce qui a effectivement été confirmé

| Élément | Résultat | Niveau de preuve |
| --- | --- | --- |
| Matériel | Module **CB3S / BK7231N**, flash **2 Mio** | Marquage du module + sauvegarde série |
| Sauvegarde | **Deux lectures complètes de 2 097 152 octets, identiques octet par octet** | Comparaison locale ; binaires conservés **hors GitHub** |
| Connexion ESPHome | Wi-Fi local fonctionnel ; **mise à jour OTA Wi-Fi réussie** | Essai utilisateur |
| Relais 1–4 | **P14, P6, P7, P8**, commande active HIGH | Commandes Web réellement observées sur les quatre sorties ; LED individuelles en cohérence |
| Bouton physique | **P10**, actif LOW, appui bref fonctionnel | Essai utilisateur : commande générale ON/OFF |
| LED visible sous le bouton | **P23**, actif LOW | Essai différentiel avec deux commandes Web indépendantes sur P11 et P23 |
| Sélection par bouton long | Temporisations jugées correctes par l'utilisateur dans une version précédente | Essai utilisateur ; la dernière version corrective **V3 n'est pas encore confirmée flashée** |
| Dernière configuration V3 | L'utilisateur indique « c'est okay » après correction des erreurs de validation | **Validation dans l'éditeur ESPHome** ; compiler, flasher et tester sur matériel restent à confirmer pour cette révision |

**Ce qui reste ouvert :** révision/ordre physique exact des prises, comportement électrique transitoire au boot, tests après coupure, autonomie sans WAN, recette répétée de chaque canal. `restore_mode: ALWAYS_OFF` exprime l'état logiciel attendu, sans démontrer à lui seul l'absence d'impulsion au démarrage.

## 2. Pourquoi l'extraction du firmware Tuya ne suffisait pas

La configuration extraite de la sauvegarde Tuya associait :

- **P14/P6/P7/P8** aux quatre relais (HIGH).
- **P10** au bouton général (LOW).
- **P23** au voyant réseau (LOW).
- **P11** à un voyant général (LOW).

**Sur l'appareil réel, la LED visible sous le bouton est pilotée par P23, et non par P11.** Nous avions d'abord attribué la LED du bouton à P11, ce qui ne produisait aucune réaction à la commande.

### La méthode qui a réellement départagé P11 et P23

1. Retirer temporairement `status_led`, qui utilisait **P23** et pouvait entrer en conflit avec une commande manuelle.
2. Ajouter **exactement deux** interrupteurs GPIO de diagnostic, noms explicites : **TEST LED GPIO P11** et **TEST LED GPIO P23**, actifs LOW.
3. Tester les deux séparément via le serveur Web.
4. Relever l'observation : **P23 commande la LED du bouton**.
5. Supprimer les interrupteurs temporaires. Déclarer **un seul propriétaire** de P23 : `output` GPIO piloté par `light` binaire. **Ne pas réintroduire `status_led` sur P23**.

**Leçon générale :** même avec un dump d'origine, distinguer la fonction logique du firmware de **l'élément physique visible**. Faire une mesure/test discriminant plutôt que réaffecter les broches au hasard.

## 3. Logique finale retenue pour un seul bouton

### Mode normal

- **Appui bref (50–800 ms)** : **si au moins un relais est ON, éteindre les quatre** ; sinon, les allumer tous.
- LED P23 **allumée seulement lorsque les quatre relais sont ON**.
- Une configuration antérieure testait par erreur « les quatre ON » avant d'éteindre : avec un seul relais ON, l'appui bref allumait les quatre. Il faut le **OU** logique, pas le **ET**.

### Mode sélection

- **Maintien pendant 2 s** : entrer en sélection **dès le seuil**, sans attendre le relâchement.
- **Appui bref** en sélection : avancer parmi les prises 1 → 2 → 3 → 4 → 1.
- **LED P23** : 1 à 4 impulsions, répétées avec une pause, pour annoncer la prise choisie.
- **Nouvel appui long (2 s)** : inverser **uniquement la prise sélectionnée**, puis sortir du mode.
- **10 s d'inactivité** : abandonner la sélection sans toucher aux relais.
- Appuis intermédiaires **0,8 à 2 s** : aucune commande intentionnelle.
- Sortie du mode sélection : la LED retrouve son rôle de témoin général.

**Leçon ESPHome :** `on_click` avec une durée longue exécute l'action **au relâchement**. Pour réagir pendant le maintien, démarrer un `script` avec `delay: 2s` sur `on_press`, puis l'arrêter sur `on_release` si le seuil n'a pas été atteint. Déclencher l'action des appuis courts à partir de `on_click`.

## 4. Erreurs de configuration rencontrées et remèdes

| Symptôme observé | Ce qui a aidé |
| --- | --- |
| 8 erreurs « Couldn't find ID 'actualiser_voyant' » | Vérifier que le **YAML complet** contient bien la déclaration du script, que son nom est identique partout et que le fichier a été **remplacé**, pas ajouté en fin de document |
| Plusieurs `sensor:`, `binary_sensor:`, GPIO ou IDs dupliqués après fusion | **Repartir de la dernière version fonctionnelle**, fusionner une fois les sections racines, supprimer les définitions obsolètes, compter les références de GPIO |
| Multiples erreurs d'IDs `mode_selection`, `relais_1`… et `voyant_general` dans l'éditeur | Les erreurs ont accompagné une réorganisation de scripts ; reconstruire une version cohérente et complète avec tous les `globals`, `switch`, `output/light`, `script` et `binary_sensor` présents, puis valider. **Ne pas conclure à une règle générale d'ordre des IDs à partir des seuls soulignements de l'éditeur** |
| « At least one of 'id:' or 'name:' is required » | Donner un **`id` et un `name` explicites** aux boutons logiciels de la version en cause, puis relancer Validate ; la correction a été acceptée par l'utilisateur |
| LED réactive au boot, mais pas aux commandes | Contrôle de la **bonne broche** et suppression de `status_led` concurrent ; finalement **P23** |
| Appui bref avec seulement 1–3 sorties ON rallumait tout | Condition `or` sur l'état des quatre relais : **tout OFF si au moins un ON**, sinon tout ON |
| Corrections répétées qui génèrent d'autres erreurs | Éviter les remaniements non nécessaires ; faire un **diff minimal**, valider, garder une sauvegarde de la version qui marchait, demander la première erreur exacte |

**Vérification recommandée avant publication ou flash :** (1) parse YAML ; (2) unicité des clés racines, identifiants et GPIO ; (3) résolution de toutes les références (`id`, `script.execute`, `switch`, `light`) ; (4) **ESPHome Validate** ; (5) **compilation** ; (6) essai OTA puis recette fonctionnelle. Un simple parse YAML **ne valide pas la configuration ESPHome**.

## 5. Fichiers, accès et sécurité

- Exemple de configuration sans identifiants : [`multiprise-tv-selection-p23.yaml`](../firmware/5225000500/multiprise-tv-selection-p23.yaml).
- La configuration d'essai est adaptée à **ESPHome 2026.10.2**, d'après l'éditeur affiché pendant le diagnostic ; les versions futures peuvent demander des adaptations.
- Authentification Web locale (HTTP Digest), API chiffrée, OTA ESPHome natif chiffré ; **`web_server.ota: false` retire volontairement l'upload HTTP dans l'interface Web**.
- Les champs `!secret` du YAML public **ne contiennent aucune valeur réelle**. Ne pas publier `secrets.yaml`, dump Tuya, clés locales, identifiants Wi-Fi, images compilées avec secrets, captures qui révèlent des credentials.
- Documenter le matériel **par modèle et révision** : la prise simple LSC 5225000301 utilise **BK7238** avec un mappage totalement différent ; ne pas lui appliquer la configuration CB3S.

## 6. Recette à terminer avant de déclarer « production »

- [x] Sauvegarde Tuya en double, identique.
- [x] Flash série initial puis OTA ESPHome Wi-Fi fonctionnel.
- [x] Commandes Web et commutation des quatre relais confirmées.
- [x] Appui bref du bouton P10 confirmé.
- [x] LED sous bouton trouvée par test différentiel : P23.
- [x] Temporisations du mode sélection jugées correctes dans une version précédente.
- [ ] Compilation **et** flash de la dernière V3 confirmés explicitement.
- [ ] Appui bref « 1–3 relais ON → tout OFF » confirmé après flash V3.
- [ ] Validation du voyant P23 en mode normal après V3.
- [ ] Validation des séquences 1–4 et de la sortie après 10 s sur V3.
- [ ] Correspondance des quatre prises avec leur position physique notée.
- [ ] Observation du comportement des relais lors de redémarrages et coupures secteur.
- [ ] Contrôle sans WAN, sans HA, avec accès local selon besoin.

## 7. Pour une future commande de porte de garage

Réutiliser **la méthode**, **pas le brochage ni la logique de puissance** :

1. Référence précise, photos/schéma, type d'alimentation et surtout nature de la commande du moteur.
2. Déterminer si le contrôleur attend une **impulsion sur contact sec** (fréquent), une entrée maintenue ou une interface propriétaire ; **ne jamais supposer que « relais ON » = « porte ouverte »**.
3. Sauvegarder le firmware original si une conversion est envisagée ; identifier le microcontrôleur avant flash.
4. Exposer une **commande momentanée** avec durée bornée si le contrôleur requiert un contact impulsionnel ; définir le comportement au boot.
5. Prévoir des **capteurs de position réels** si l'on souhaite afficher « ouverte/fermée ». Une simple sortie relais ne renseigne pas l'état de la porte.
6. Préserver les dispositifs anti-écrasement, cellules photoélectriques et fins de course ; interdire toute automatisation dangereuse sans confirmation de leur fonctionnement.
7. Construire une version minimale, isoler GPIO et LED via commandes de diagnostic, valider dans ESPHome et tester chaque geste.

**Règle de travail tirée de ce projet :** une étape à la fois, sauvegarder la base qui fonctionne, ne modifier que ce qui est observablement défectueux, distinguer les hypothèses des résultats confirmés.

## Historique / sources

- [Ticket de suivi multiprise #4](https://github.com/ocverclock/action-lsc-esphome/issues/4).
- [Procédure série et sauvegarde](09-multiprise-uart-esphome.md).
- [Modèle matériel](../models/5225000500.md).
- [Démontage de la variante Action 3202887 (Elektroda)](https://www.elektroda.com/rtvforum/topic4095176.html).
- [CB3S / LibreTiny](https://docs.libretiny.eu/boards/cb3s/).
