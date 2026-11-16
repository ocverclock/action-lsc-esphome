# Phase finale — Multiprise Action / LSC à quatre prises

> **Mise à jour du 16/11/2026 :** les sections datées du 14/11 ci-dessous décrivent le **plan historique avant conversion**. Depuis, la multiprise française 5225000500 est identifiée **CB3S/BK7231N**, sauvegardée en double et **flashée sous ESPHome** ; Wi-Fi/OTA, quatre relais P14/P6/P7/P8, bouton P10 et LED du bouton P23 ont été confirmés. La dernière V3 est annoncée validée dans l'éditeur, mais sa recette après flash et les tests boot/WAN restent à réaliser. Voir [le retour d'expérience actualisé](10-retour-experience-multiprise-5225000500.md) et le [YAML V3 sans secrets](../firmware/5225000500/multiprise-tv-selection-p23.yaml).

**Modèle recensé : LSC 5225000500, « Multiple TV », quatre canaux, sans compteur** (d'après l'inventaire Tuya réalisé le 11 novembre 2026).

**Statut au 14/11/2026 : non convertie.** Les quatre sorties sont exposées comme **DP1, DP2, DP3, DP4 dans Tuya**. Cela **ne permet pas de déduire les broches GPIO, le chipset, les niveaux actifs, ni l'ordre physique** des quatre prises.

## Pourquoi elle passe en dernier

Un mauvais flash ou un mauvais mappage de relais peut actionner une sortie involontairement. Les quatre sorties impliquent aussi des tests de sûreté supplémentaires. La réussite de la prise pilote BK7238 **n'établit pas** la compatibilité de la multiprise.

## Plan de qualification **avant flash**

1. Photographier étiquette complète (référence exacte, EAN, puissance admissible), relever versions firmware principal/MCU dans Tuya ; conserver ces relevés **en privé**.
2. Identifier matériellement (documentation de même révision, dépôt communautaire ou inspection par un professionnel hors tension) : chipset, module Wi‑Fi, mémoire flash, bootloader, éventuellement MCU secondaire et protocole de commande.
3. Déterminer si un OTA de type UnTuyaOS3 **est réellement compatible**. Sinon, choisir une autre méthode adaptée ; **ne pas forcer la plateforme T1**.
4. Obtenir le mappage **GPIO → sortie physique 1/2/3/4** et vérifier polarité, bouton(s), LED(s), éventuel MCU chargé des relais.
5. Prévoir la sauvegarde et la stratégie de récupération avant écriture ; pas de charge branchée.
6. Définir un firmware **fail-safe**, démarrage toutes sorties OFF **si le matériel le permet**, et aucun basculement imprévu pendant boot/OTA.
7. Valider la version ESPHome compilée **avant** de flasher une seule multiprise.

## Plan de recette après conversion

- [ ] Les **quatre** sorties sont désactivées après premier démarrage (sans charge).
- [ ] Chacune peut être activée **individuellement** ; aucune autre ne change d'état.
- [ ] L'ordre des sorties virtuelles correspond à leur position physique.
- [ ] Chaque bouton physique (s'il existe) n'agit que sur la bonne sortie, sans auto-activation inattendue.
- [ ] Redémarrage, coupure secteur, reconnexion Wi‑Fi et restauration d'état sûrs sur **chacun des quatre canaux**.
- [ ] Intégration Home Assistant : quatre interrupteurs distincts et leurs retours d'état.
- [ ] Réseau WAN bloqué : **10 cycles ON/OFF par sortie**, sans dépendance Tuya.
- [ ] Aucun relais ne commute involontairement pendant boot ni récupération du Wi‑Fi.

## Ce qu'il ne faut surtout pas faire

- Ne **pas** copier la configuration P24/P26/P9/P11 de la prise simple.
- Ne **pas** assimiler DP1–DP4 aux broches P1–P4.
- Ne **pas** considérer le même fabricant comme preuve du même bootloader.
- Ne **pas** brancher une TV, un amplificateur ou une charge réelle pendant l'identification et les premiers tests.
- Ne **pas** laisser des sorties de puissance dans un état indéterminé si le Wi‑Fi/API tombe.

## Prochaine décision

Le [modèle 5225000500](../models/5225000500.md) reste en **inventaire matériel** tant que le chipset et les relais ne sont pas identifiés. Une fois les prises simples éprouvées **sans Internet**, créer une configuration spécifique pour la multiprise et la soumettre à la recette par canal. **Aucune image OTA ni YAML de production n'est proposée à ce stade** : ce serait inventer des paramètres dangereux.
