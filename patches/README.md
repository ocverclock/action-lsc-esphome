# Correctif DHCP / NetworkManager pour UnTuyaOS3

**Révision amont ciblée : `14b56c3` du 29/10/2026.** Correctif dérivé des essais réussis du 11/11/2026 sur iMac Linux avec une interface Wi‑Fi `wls1` gérée par NetworkManager. **Il ne faut pas l'appliquer automatiquement à une version UnTuyaOS3 plus récente.**

Fichier : [`untuyaos3-14b56c3-networkmanager.patch`](untuyaos3-14b56c3-networkmanager.patch).

## Symptômes observés

- UnTuyaOS3 voit un AP Tuya ouvert `SmartLife-XXXX` et s'y associe.
- `dhclient` reçoit un `DHCPACK` et une IPv4 sur `192.168.176.0/24`.
- Cependant `connect.sh` affiche `DHCP did not assign an address` : `dhclient -x` a supprimé l'IPv4 avant son contrôle.
- En empêchant `dhclient -x`, une seconde anomalie subsiste : l'ancienne adresse du réseau habituel reste affichée avec l'adresse Tuya, et le script peut lire la mauvaise.

## Solution validée expérimentalement

1. Après `iw dev "$iface" connect -w "$match"`, nettoyer **uniquement l'IPv4** obsolète de l'interface Wi‑Fi avec `ip -4 addr flush dev "$iface"`.
2. Acquérir l'adresse Tuya par `dhclient -1` puis **ne pas appeler `dhclient -x` avant `ap-ota.py`**.
3. À la restitution de l'interface, **terminer le client DHCP et vider l'IPv4 Tuya** juste avant `nmcli dev set ... managed yes` puis reconnecter normalement.

Dans le test de connexion **sans flash**, on obtenait une IPv4 unique `192.168.176.100/24` et ping vers `192.168.176.1` **2/2 réponses**. Le même correctif a ensuite permis la véritable OTA avec **432292/432292 octets envoyés**, `Reported OTA finished!`, et un Kickstart ESPHome **démarré**.

## Application reproductible

Faire une copie du dépôt original, puis appliquer **uniquement si le commit et l'environnement concordent** :

```bash
git clone https://github.com/tuya-cloudcutter/UnTuyaOS3.git ~/UnTuyaOS3
git clone https://github.com/ocverclock/action-lsc-esphome.git ~/action-lsc-esphome
cd ~/UnTuyaOS3
git checkout 14b56c3
git status --short
# Avant de toucher aux scripts, sauvegarder les fichiers.
mkdir -p ~/UnTuyaOS3-sauvegarde
cp -a scripts/connect.sh scripts/restore-network.sh ~/UnTuyaOS3-sauvegarde/
git apply --check ~/action-lsc-esphome/patches/untuyaos3-14b56c3-networkmanager.patch
git apply ~/action-lsc-esphome/patches/untuyaos3-14b56c3-networkmanager.patch
bash -n scripts/connect.sh
bash -n scripts/restore-network.sh
git diff --check
git --no-pager diff --stat
git --no-pager diff -- scripts/connect.sh scripts/restore-network.sh
```

**Important :** ces commandes **n'effectuent aucun flash**. Après application, valider l'association et le DHCP **dans un test isolé du script de connexion**, en veillant à restaurer le réseau via `restore-network.sh`. Le script principal `UnTuyaOS3.sh` **enchaîne automatiquement avec l'écriture OTA** après connexion réussie, donc ne pas l'utiliser pour un diagnostic réseau non destructif.

### Retour en arrière (uniquement si aucun flash en cours)

```bash
cd ~/UnTuyaOS3
git checkout -- scripts/connect.sh scripts/restore-network.sh
```

Si des modifications locales avaient été présentes **avant** l'application, ne pas utiliser cette commande aveuglément : restaurer plutôt les copies sauvegardées. Ne jamais interrompre un transfert actif pour « restaurer ».

## Limites et précautions

- Le correctif **ne teste pas** une prise inconnue et ne protège pas d'une image incompatible ou d'un « brick ».
- Le patch est basé sur **deux scripts exacts d'un commit figé** ; `git apply --check` doit réussir avant `git apply`.
- La désactivation de `dhclient -x` avant le transfert conserve temporairement le client DHCP ; c'est volontaire, mais il doit être arrêté à la restitution de l'interface.
- Si l'hôte utilise `dhcpcd` ou `udhcpc` plutôt que `dhclient`, **ce correctif n'a pas été qualifié** sur cet environnement.
- Si UnTuyaOS3 corrige upstream ce problème dans une prochaine version, **préférer la solution maintenue**.

La suite complète figure dans [l'étape 2 — Flash UnTuyaOS3](../docs/02-flash-untuyaos3.md).

[Sources originales `connect.sh`](https://github.com/tuya-cloudcutter/UnTuyaOS3/blob/14b56c3/scripts/connect.sh) · [`restore-network.sh`](https://github.com/tuya-cloudcutter/UnTuyaOS3/blob/14b56c3/scripts/restore-network.sh).
