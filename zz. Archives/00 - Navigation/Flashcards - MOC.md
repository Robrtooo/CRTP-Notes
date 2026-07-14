# Flashcards - MOC

> [!summary] Révision active — 60 cartes, 7 decks
> Page centralisée des flashcards. Chaque carte est un **toggle `[!question]-`** : lis la question, réponds de tête, puis déplie pour vérifier. Format léger, rien à installer.

## Decks par thème

| Thème | Deck | Cartes | Cours |
|---|---|---|---|
| Raisonnement | [[Flashcards - Attack flow]] | 8 | [[2. Attack flow]] |
| AD — fondamentaux | [[Flashcards - Fondamentaux AD]] | 8 | [[Fondamentaux AD]] |
| AD — attaques | [[Flashcards - Attaques AD]] | 8 | [[Attaques AD]] |
| Kerberos | [[Flashcards - Kerberos]] | 10 | [[Kerberos]] |
| Énumération | [[Flashcards - Énumération]] | 8 | [[Énumération AD]] |
| PowerShell | [[Flashcards - PowerShell]] | 10 | [[What is PowerShell]] |
| Offensive .NET | [[Flashcards - Offensive .NET]] | 8 | [[Offensive .NET]] |

## Tous les decks (auto)

Liste générée automatiquement — tout nouveau deck y apparaît.

```dataview
LIST
FROM "03 - Cours"
WHERE contains(file.name, "Flashcards")
SORT file.name ASC
```

## Comment ça marche

- **Réviser** : ouvre un deck, lis chaque question, réponds de tête, puis clique le toggle pour déplier la réponse.
- **Ajouter une carte** : `> [!question]- Ma question ?` puis la réponse sur la ligne suivante préfixée de `> `. Une ligne vide entre deux cartes.
- **Nouveau deck** : créer une note `Flashcards - X` et suivre le même format de toggles.

## Voir aussi

- [[Cours - MOC]]
- [[1. CRTP - Index]]
