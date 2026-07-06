# Flashcards - MOC

> [!summary] Révision active — 60 cartes, 7 decks
> Page centralisée des flashcards. Elles sont au format Spaced Repetition : le plugin planifie automatiquement les révisions (algorithme SM-2). Lance la commande **« Spaced Repetition: Review flashcards »** (palette `Ctrl+P`) ou clique l'icône dans la barre latérale pour réviser la file du jour. Chaque deck porte le tag `#flashcards/<thème>`.

## Decks par thème

| Thème | Deck | Cartes | Cours |
|---|---|---|---|
| Raisonnement | [[Flashcards - Attack flow]] | 8 | [[Attack flow]] |
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

- **Réviser** : la commande ouvre la file du jour ; réponds *Bon / Difficile / À revoir* → le plugin espace ou rapproche la prochaine échéance.
- **Ajouter une carte** : dans un deck, écris `**Question**`, une ligne contenant seulement `?`, puis la réponse — chaque carte séparée par une ligne vide.
- **Nouveau deck** : créer une note `Flashcards - X` avec le tag `#flashcards/x` en tête.

## Voir aussi

- [[Cours - MOC]] · [[CRTP - Index]]
