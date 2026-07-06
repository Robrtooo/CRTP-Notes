# Cours - MOC

> [!summary] Résumé
> Carte des cours : la couche « comprendre le pourquoi », en amont des cheatsheets (le comment) et des flashcards (le rappel). Pivot de raisonnement : [[Attack flow]].

## Raisonnement

- [[Attack flow]] — ce que j'ai → ce que je peux en faire
- [[Flashcards - Attack flow]]

## Active Directory

- [[Fondamentaux AD]] · [[Flashcards - Fondamentaux AD]]
- [[Attaques AD]] · [[Flashcards - Attaques AD]]
- [[Kerberos]] · [[Flashcards - Kerberos]]
- Notes brutes : [[General Notes]] · [[Components of AD]]

## Énumération

- [[Énumération AD]] · [[Flashcards - Énumération]]
- Notes brutes : [[Enumeration]] · [[CheetSheet]]

## PowerShell

- [[What is PowerShell]] · [[Flashcards - PowerShell]]
- Notes brutes : [[2. Good practices]] · [[Using tools]]

## Offensive .NET

- [[Offensive .NET]] · [[Flashcards - Offensive .NET]]
- Notes brutes : [[1. General Notes]] · [[AV BYPASS]] · [[Assume Breach]]

## Tous les cours (auto)

```dataview
LIST
FROM "03 - Cours"
WHERE !contains(file.name, "Flashcards")
SORT file.folder ASC, file.name ASC
```

## Voir aussi

- [[CRTP - Index]] · [[Cheatsheets - MOC]] · [[Attaques AD - MOC]] · [[Active Directory - MOC]]
