# Icare -> Consigne 2 : Analyse du paquet

## Objectif

Identifiez le fichier modifié, avec son chemin tel qu'il figure dans `SHA256SUMS`, puis relevez les deux valeurs écrites en dur dans le code ajouté, l'identifiant numérique d'opérateur en hexadécimal et le jeton.

## Éléments reçus

Les éléments fournis sont les suivants : 
- R.A.S

## Démarche

Le paquet est une archive `.tar.gz`. Création d’une copie de travail dans un dossier :

```sh
tar -tzf icare/update-SAT-ALPHA-3-4.7.2.tar.gz
mkdir -p icare/analyse/consigne-02
tar --extract --gzip --file=icare/update-SAT-ALPHA-3-4.7.2.tar.gz --directory=icare/analyse/consigne-02 --no-same-owner --no-same-permissions
```

Depuis la racine du projet, vérifier les fichiers :

```sh
cd icare/analyse/consigne-02/update-SAT-ALPHA-3-4.7.2
sha256sum -c SHA256SUMS
```

Résultat confirmé : dix fichiers `OK`, un fichier `FAILED` : `src/tcuplink/auth.py`.

Lire le fichier sans l’exécuter :

```sh
nl -ba src/tcuplink/auth.py
```

## Conclusion

- Chemin exact figurant dans `SHA256SUMS` : `src/tcuplink/auth.py`.
- Ligne 27 : `_MAINT_OPERATOR = 0xA5`.
- Ligne 28 : `_MAINT_TOKEN = bytes.fromhex("9e3ab7f04c118d26")`.
- Lignes 32–33 : si l’identifiant correspond et que les huit premiers octets du payload correspondent au jeton, la fonction renvoie immédiatement `True`.

Le flag combine le chemin exact du fichier non conforme tel qu’il figure dans `SHA256SUMS` (`src/tcuplink/auth.py`), l’identifiant opérateur en hexadécimal (`0xA5`), puis le jeton hexadécimal écrit en dur (`9e3ab7f04c118d26`). Ces trois valeurs sont séparées par `:` et encadrées par `RM{}` :

```text
RM{src/tcuplink/auth.py:0xA5:9e3ab7f04c118d26}
```

**Flag final de l’activité.**
