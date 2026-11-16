# Icare -> Consigne 5 : tâches mises en file par l’intrus

## Objectif

Identifiez les tâches mises en file par l'intrus sur l'orchestrateur et bloquez-les toutes. Le bon blocage vous donne le flag.

## Éléments reçus

Les éléments fournis sont les suivants : 
- Console de l'orchestrateur

## Démarche

### 1. Utiliser l’indicateur issu de l’analyse précédente

Dans l’implant d’ALPHA-3, l’opérateur spécial est `0xA5`. Chercher les ouvertures de session utilisant cet opérateur permet de repérer deux événements particulièrement pertinents, tous deux avec `auth=maint` :

```text
2026-11-10T22:41:07Z event=session.open sid=7f3a9c op=0xA5 src=203.0.113.41 auth=maint
2026-11-11T00:02:11Z event=session.open sid=6e0b93 op=0xA5 src=203.0.113.77 auth=maint
```

### 2. Suivre la session plutôt que le seul opérateur

Le champ `sid` relie les événements d’une même session. L'acteur malveillantt ouvre avec `op=0xA5`, puis certaines tâches de ces mêmes sessions portent `op=0x11` ou `op=0x21`.

Un filtre limité aux lignes contenant `op=0xA5` manquerait donc trois tâches : `DK-7432`, `DK-2372` et `DK-1075`. Il faut d’abord identifier les sessions suspectes, puis suivre tous leurs événements, indépendamment du champ opérateur de chaque ligne.

Les sessions à suivre sont `7f3a9c` et `6e0b93`. Leurs fermetures sont respectivement à 22:58:12 UTC et 00:09:40 UTC. Les événements relevés sont situés entre leurs ouvertures et fermetures.

### 3. Retenir les événements qui créent une tâche

Sélectionner les événements `event=job.queue` associés aux deux sessions. Le champ `job` fournit l’identifiant à utiliser dans l’interface de blocage. Les champs `target`, `build`, `station` et `window` décrivent la tâche.

## Conclusion

**Tâches à bloquer :** `DK-8072`, `DK-4846`, `DK-8973`, `DK-7432`, `DK-2372`, `DK-1075`.

Le flag est délivré par la console après le blocage correct des six tâches. la valeur retournée par la console est reprise telle quelle, avec son encadrement `RM{}`.

**Flag obtenu :**

```text
RM{0301aeef7e2231e9a88be73a5dd34ffae8daaf9a}
```

**Flag final de l’activité.**