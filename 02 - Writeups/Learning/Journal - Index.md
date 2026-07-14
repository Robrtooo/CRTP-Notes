# Journal - Index

> [!summary] Objectif
> Journal de lab CRTP : un fichier par Learning Objective. À remplir **au fil du lab** — objectif, la commande qui a réellement marché, et le piège rencontré. C'est ce qui transforme le lab en connaissance durable et enrichit les cheatsheets.

> [!tip] Workflow
> 1. Ouvre le LO en cours. 2. Suis la fiche liée + [[Attack flow]]. 3. Note la commande qui marche et le piège. 4. Passe `statut` à `en cours` puis `validé` dans le frontmatter. 5. Si tu découvres un vrai truc, reporte-le aussi dans la cheatsheet concernée.

## Progression

```dataview
TABLE statut AS "Statut"
FROM "02 - Writeups/Learning"
WHERE lo
SORT lo ASC
```

## Comment marquer l'avancement

Dans chaque note, change le champ `statut:` du frontmatter :
- `à faire` — pas encore commencé
- `en cours` — en train de le faire
- `validé` — LO réussi et noté

## Voir aussi

- [[Index]]
- [[Attack flow]]
- [[Checklist - Méthodologie CRTP]]
