# Icare -> Consigne 1 : Contrôle d'intégrité

Session : 14 novembre 2026.

## Objectif

Calculez l'empreinte réelle du paquet livré et confirmez l'écart avec le manifeste signé.

## Éléments reçus

- `icare/manifest.json`
- `icare/update-SAT-ALPHA-3-4.7.2.tar.gz`

## Démarche

1. Lire le manifeste :

```sh
cat icare/manifest.json
```

Le manifeste contient plusieurs paquets. Prendre l’entrée dont `name` correspond exactement à `update-SAT-ALPHA-3-4.7.2.tar.gz` : cible `SAT-ALPHA-3`, révision `4.7.2`. Le champ `digest.algorithm` indique `sha256`.

2. Calculer l'empreinte du paquet fournit (signalé par le controle qualité) :

```sh
sha256sum icare/update-SAT-ALPHA-3-4.7.2.tar.gz
```

3. Comparer le SHA-256 calculé sur le paquet `icare/update-SAT-ALPHA-3-4.7.2.tar.gz` complet à la valeur `digest.value` fournie dans `icare/manifest.json` dont le champ `name` désigne exactement ce paquet. La comparaison porte sur l’archive compressée elle-même, pas sur le dossier extrait ni sur ses fichiers internes.

| Mesure | Manifeste | Fichier reçu |
|---|---|---|
| SHA-256 | `9310f4c26ef83e56fd569ab8e3afd9e6cc959e6b9037d2a389ead0abada0bff8` | `65609e9a9e7710b8c1a1a9b2d56d7df2f3964eadf67edb2333103298c0e3eccf` |
| Taille en octets | 2943 | 15163 |

## Conclusion

Le SHA-256 du paquet `.tar.gz` reçu diffère de l’empreinte de référence indiquée dans son entrée du manifeste : **le paquet ne correspond pas à la version attendue**.

Le flag contient le SHA-256 réellement calculé sur le paquet fournit signalé par le controle qualité, encadré par `RM{}`.

```text
RM{65609e9a9e7710b8c1a1a9b2d56d7df2f3964eadf67edb2333103298c0e3eccf}
```

**Flag final de l’activité.**

