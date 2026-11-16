# Icare -> Consigne 4 : Identification d'IOC

## Objectif

Via le scanner, identifiez tous les paquets du lot porteurs de l'implant, et listez leurs cibles par ordre alphabétique.

## Éléments reçus

Les éléments fournis sont les suivants : 
- Scanner Yara du déppot

## Point de départ

L’implant connu dans `src/tcuplink/auth.py` accepte immédiatement certains ordres avec `return True` avant les contrôles normaux. Les constantes connues sont `0xA5` et `9e3ab7f04c118d26`.

## Démarche

| Étape | Critère utilisé | Résultat et enseignement |
|---|---|---|
| 1 | `verify_command`, opérateur, payload, acceptation et mention de maintenance | SAT-ALPHA-3 détecté ; trop lié aux noms et commentaires de l’exemple. |
| 2 | Fonction connue, opérateur, payload, `return True`, HMAC et compteur | Quatre correspondances signalées, sans liste fournie à ce stade. |
| 3 | Constantes hexadécimales au format connu ou condition opérateur/payload immédiatement suivie de `return True` avant registre/HMAC | Seulement ALPHA-3 signalé avec ces règles, contraintes de forme trop fortes pour couvrir les variantes. |
| 4 | Règle combinée : acceptation avant registre/HMAC, OU opérateur + payload + acceptation + constante | **Cinq cibles :** GS-SUD, SAT-ALPHA-3, SAT-BETA-1, SAT-BETA-2, SAT-DELTA-2. Flag correspondant refusé. La branche fondée sur les seules constantes peut détecter du code non malveillant. |
| 5 | Condition sur `operator_id` sur une ligne, immédiatement suivie d’un `return True`, avant registre, avec payload et valeur hexadécimale | **Trois cibles :** SAT-ALPHA-3, SAT-BETA-1, SAT-DELTA-2. Flag correspondant refusé. La règle impose trop de détails syntaxiques. |
| 6 | `return True` avant registre OU comparaison HMAC OU rejet `AuthError`, sans imposer fonction, jeton ou condition sur une ligne | **Quatre cibles :** GS-SUD, SAT-ALPHA-3, SAT-BETA-1, SAT-DELTA-2. Flag accepté, validé sur la plateforme. |

## Règle ayant conduit à la validation

Copie réutilisable : `icare/scripts/04_search.yar`.

```yara
rule search
{
    strings:
        $accept = /return[ \t]+True\b/                              // Détecte une instruction return True

        $check_registry = "REGISTRY.get("                           // Détecte un accès au registre
        $check_hmac = "hmac.compare_digest("                        // Détecte une comparaison cryptographique HMAC
        $check_reject = /raise[ \t]+AuthError[ \t]*\(/              // Détecte une exception de rejet d'authentification

    condition:
        $accept and                                            // Vérifie la présence d'un return True
        (
            ($check_registry and @accept < @check_registry)         // Vérifie si return True apparaît avant l'accès au registre
            or
            ($check_hmac and @accept < @check_hmac)                 // Vérifie si return True apparaît avant la vérification HMAC
            or
            ($check_reject and @accept < @check_reject)             // Vérifie si return True apparaît avant le rejet d'authentification
        )
}
```

## Explication ligne par ligne

Les numéros ci-dessous correspondent exactement au fichier `.yar`, lignes vides comprises.

| Ligne | Code / rôle | Explication |
|---|---|---|
| 1 | `rule search` | Déclare une règle nommée `search`. Ce nom identifie le verdict ; il ne change pas les motifs recherchés. |
| 2 | `{` | Ouvre le corps de la règle. |
| 3 | `strings:` | Ouvre la liste des motifs recherchés dans chaque fichier analysé. |
| 4 | `$accept = /return[ \t]+True\b/` | Définit une expression régulière cherchant `return`, au moins un espace ou une tabulation, puis `True` avec une frontière de mot après celui-ci. Cette frontière évite par exemple `TrueValue`. Le motif est sensible à la casse. |
| 5 | Ligne vide | Sépare visuellement les motifs ; aucun effet sur le verdict. |
| 6 | `$check_registry = "REGISTRY.get("` | Recherche cette chaîne littérale exacte, parenthèse comprise : indice de consultation du registre des opérateurs. |
| 7 | `$check_hmac = "hmac.compare_digest("` | Recherche cette chaîne littérale exacte : indice d’une comparaison d’authentificateur. |
| 8 | `$check_reject = /raise[ \t]+AuthError[ \t]*\(/` | Recherche `raise`, au moins un espace/tabulation, `AuthError`, éventuellement des espaces/tabulations, puis une parenthèse ouvrante littérale. C’est un indice de rejet d’authentification. |
| 9 | Ligne vide | Séparation visuelle. |
| 10 | `condition:` | Ouvre l’expression logique qui détermine si le fichier correspond à la règle. |
| 11 | `$accept and` | Exige la présence du motif d’acceptation ET la satisfaction du groupe de conditions suivant. |
| 12 | `(` | Ouvre le groupe de trois possibilités alternatives. |
| 13 | `($check_registry and @accept < @check_registry)` | Exige la présence du motif de registre et que la première acceptation apparaisse avant sa première occurrence. |
| 14 | `or` | Une autre possibilité peut suffire à satisfaire le groupe. |
| 15 | `($check_hmac and @accept < @check_hmac)` | Exige le motif de comparaison HMAC et une première acceptation située avant sa première occurrence. |
| 16 | `or` | Introduit la troisième possibilité. |
| 17 | `($check_reject and @accept < @check_reject)` | Exige le motif de rejet et une première acceptation située avant sa première occurrence. |
| 18 | `)` | Ferme le groupe : au moins une des trois possibilités doit être vraie. |
| 19 | `}` | Ferme la règle. |

## Conclusion

| Cible | Version rapportée | Fichier détecté |
|---|---|---|
| GS-SUD | 2.1.0 | `src/station/ctl.py` |
| SAT-ALPHA-3 | 4.7.2 | `src/tcuplink/auth.py` |
| SAT-BETA-1 | 3.4.1 | `src/adcs/ctl.py` |
| SAT-DELTA-2 | 5.1.2 | `src/payload/sched.py` |

Le flag contient uniquement les noms des quatre cibles identifiées, triés par ordre alphabétique, séparés par `:` et encadrés par `RM{}` :

```text
RM{GS-SUD:SAT-ALPHA-3:SAT-BETA-1:SAT-DELTA-2}
```

**Flag final de l’activité.**
