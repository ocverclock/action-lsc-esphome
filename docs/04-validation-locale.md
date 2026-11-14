# Étape 4 — Recette d'acceptation « sans Internet »

**Objectif :** démontrer que l'appareil reste pilotable avec une **connexion Wi‑Fi et un réseau local**, mais **sans accès aux serveurs Tuya ni à Internet**. La conversion de la prise pilote a réussi ; le test WAN isolé **n'était pas encore documenté comme réussi** au 14/11/2026.

## Architecture attendue

```text
Navigateur / application Home Assistant locale
                 |
             LAN / UniFi
                 |
       Home Assistant (API ESPHome)
                 |
        VLAN/SSID IoT local
                 |
       ESPHome BK7238 → relais
```

Internet peut tomber sans supprimer le fonctionnement local. En revanche **couper la borne Wi‑Fi, le routeur LAN ou Home Assistant** peut supprimer la commande distante ; le **bouton physique** doit encore fonctionner. Utiliser un accès local à HA, pas un nom de domaine qui nécessiterait Internet.

## Tests à réaliser — puis cocher

### A. Sécurité et fonctions de base

- [ ] Pas de charge au premier démarrage ; aucun échauffement anormal.
- [ ] Relais OFF/ON depuis l'interface Web locale.
- [ ] Appui physique bref : bascule relais.
- [ ] LED de relais suit bien le relais ; état à l'allumage maîtrisé.
- [ ] Redémarrage logiciel sans perte du réseau ni de l'accès OTA.
- [ ] Coupure/remise sous tension secteur **prise seule** et contrôle des trois politiques de restauration, sur charge non critique.
- [ ] Retour à un réseau de secours possible si erreur SSID/mot de passe.

### B. Home Assistant sur LAN

- [ ] Intégration ESPHome par API, entité relais présente dans HA.
- [ ] Une action HA ON/OFF change réellement l'état physique.
- [ ] Retour d'état cohérent après appui physique et redémarrage.
- [ ] Après bascule de réseau/VLAN, HA retrouve l'appareil à sa nouvelle IP.
- [ ] Si VLAN séparé, pare-feu autorise HA ↔ appareil (API ESPHome, typiquement TCP 6053) et les règles nécessaires de découverte ; sinon ajout manuel par adresse IP.
- [ ] HA reste accessible **localement** (IP/adresse LAN), sans service cloud.

### C. Preuve de fonctionnement sans Internet

1. **Conserver le LAN et le Wi‑Fi actifs.** Ne pas couper au disjoncteur ni désactiver le SSID IoT.
2. Dans UniFi/le routeur, **bloquer temporairement l'accès WAN de la prise test seulement**, ou couper le WAN global uniquement si cela ne perturbe pas les autres équipements du domicile.
3. Depuis un appareil sur le LAN, **ouvrir Home Assistant par son adresse locale** et actionner le relais **10 fois** (alternance ON/OFF) ; vérifier physiquement les dix transitions et les retours d'état.
4. Actionner le bouton physique puis regarder si l'état HA se met à jour.
5. Redémarrer uniquement la prise et répéter le test ; attendre la reconnexion Wi‑Fi.
6. Vérifier qu'aucun appel à l'application Tuya, aucun DNS public et aucun serveur extérieur n'est requis pour piloter la prise.
7. Restaurer l'accès WAN et les règles de sécurité habituelles.

**Critère d'acceptation :** 10/10 ordres locaux corrects, bouton indépendant du WAN, reconnexion après redémarrage, HA et interface Web opérationnels pendant le blocage WAN. Noter **PASS/FAIL** et la date dans [la fiche modèle](../models/5225000301.md) ; ne pas déduire la réussite du seul fait qu'ESPHome est installé.

### D. Métrologie (non validée tant que référence absente)

- [ ] Tension plausible au repos, comparée à une référence adaptée.
- [ ] À vide, courant et puissance proches de zéro.
- [ ] Charge résistive stabilisée ≥ 2 minutes : noter U, I, P et wattmètre indépendant.
- [ ] Deuxième charge : mêmes relevés.
- [ ] Vérification de cohérence `P ≤ U×I` (tolérance d'acquisition/asynchronisme).
- [ ] Comparaison avant/après correction du compteur BK72xx avec **version ESPHome notée**.
- [ ] Énergie (kWh) suivie sur durée connue, sans dérive manifeste.

Les mesures erronées **n'empêchent pas nécessairement le pilotage local** : distinguer la validation du **relais** de celle du **compteur**.

## Fiche d'essai à recopier pour chaque prise

```text
Référence / révision :
Date :
Version Tuya d'origine :
Chipset confirmé / source :
Firmware / version ESPHome :
Réseau local/VLAN (sans credentials) :
Relais : PASS / FAIL
Bouton : PASS / FAIL
Restauration secteur : PASS / FAIL
API Home Assistant : PASS / FAIL
WAN bloqué (10 ordres) : PASS / FAIL / NON TESTÉ
Mesures BL0937 : VALIDÉES / EXPÉRIMENTALES / ABSENTES
Récupération OTA disponible : OUI / NON
Observations et corrective :
```

**Passage à une deuxième prise :** relais, bouton, réseau et test WAN doivent être **PASS**. Métrologie peut rester marquée *expérimentale*, mais ses valeurs ne doivent pas servir à des décisions automatiques. La multiprise exige une validation supplémentaire **pour chaque canal**.
