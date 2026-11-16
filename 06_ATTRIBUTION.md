# Icare -> Consigne 6 : Attribution

Date de travail : 15 novembre 2026.

## Objectif

Le flag porte l'heure UTC de la connexion VPN frauduleuse, telle qu'elle figure dans le journal du concentrateur, puis l'adresse de l'expéditeur du message d'hameçonnage qui l'a précédée.

## Éléments reçus

Les fichiers fournis :
- `icare/vpn-c.roussel.log`
- `icare/mailbox-c.roussel.zip`

## Démarche

### 1. Retrouver le mail dans les fichiers EML

J’ai exécuté, depuis `icare/analyse/mailbox-c.roussel/` :

```sh
grep -nr "VPN" eml
```

`-n` affiche les numéros de ligne et `-r` parcourt récursivement le dossier. Deux messages ressortent : `inbox-035.eml` (alerte d’échec OTP) et `inbox-076.eml` (renouvellement urgent du certificat VPN). La présence du mot VPN ne suffit pas : le contenu et les en-têtes du message 76 ont permis de retenir le message.

Son expéditeur est `it-support@segrnent-sol.aae`. Le remplacement visuel de `m` par `rn` dans le domaine (typosquatting) + la menace de blocage et la demande d’identifiants et d’OTP constituent les indices principaux.

### 2. Choisir le repère temporel et normaliser en UTC

Le champ `Date` du mail vaut `Thu, 05 Nov 2026 20:09:44 +0000`, soit `2026-11-05T20:09:44Z`. Le `+0000` et le `Z` indiquent UTC.

### 3. Filtrer les événements postérieurs au message

J’ai exécuté depuis le dossier contenant le journal :

```sh
awk '$1 >= "2026-11-05T20:09:44Z"' vpn-c.roussel.log
```

`$1` représente le premier champ de la ligne, l’horodatage. Ici les dates ont toutes le même format. Le filtre conserve aussi les jours suivants.

### 4. Distinguer tentative et connexion acceptée

La sortie de ma commande contient :

```text
2026-11-05T20:31:00Z vpn-01 sslvpn[2210]: auth.start user=c.roussel src=192.0.2.88 method=password+otp
2026-11-05T20:31:05Z vpn-01 sslvpn[2210]: auth.ok user=c.roussel src=192.0.2.88 method=password+otp client=OpenConnect/9.17 tunnel=tun23 assigned=10.66.0.125
```

`auth.start` indique le début de la tentative. `auth.ok` établit l’authentification réussie et l’attribution du tunnel. L’heure à retenir pour le flag est donc **20:31:05 UTC**, et non 20:31:00.

**Cette connexion est acceptée après la réception du mail**. L’adresse source `192.0.2.88` et le client `/9.17` diffèrent des deux sessions précédentes du même jour (`198.51.100.140`, `/9.12`).

### 5. Suivre le bon tunnel et vérifier la couverture du commit

La fermeture associée est :

```text
2026-11-05T23:48:19Z vpn-01 sslvpn[2210]: session.close user=c.roussel tunnel=tun23 duration=11834s rx=194MB tx=33MB reason=client-request
```

L’identifiant `tun23` fait le lien avec l’ouverture.

La durée correspond bien à l’intervalle de temps pendant lequel l'utilisateur malveillant réalise les modifications sur les fichiers précédemment identifés. 

### 6. Conclusion

Le flag prend l’horodatage UTC complet de la connexion acceptée (`auth.ok`), exactement tel qu’il figure dans le journal VPN (`2026-11-05T20:31:05Z`), l’adresse du champ `From` du phishing (`it-support@segrnent-sol.aae`). Les deux valeurs sont séparées par `:` et encadrées par `RM{}` :

```text
RM{2026-11-05T20:31:05Z:it-support@segrnent-sol.aae}
```

**Flag final de l’activité.**
