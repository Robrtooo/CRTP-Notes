# Conventions du vault

> [!summary] Résumé
> Conventions de structure, nommage, Markdown, images et Git pour garder le vault CRTP cohérent et navigable.

> [!warning] Cadre d'utilisation
> À utiliser uniquement dans un environnement de lab autorisé, dans le cadre de l'apprentissage et de la préparation CRTP.

## Nommage

- Préférer des titres explicites orientés apprentissage : `Cheatsheet - Sujet`, `Synthèse - Sujet`, `Checklist - Sujet`.
- Ne pas renommer brutalement les notes historiques sans validation si cela risque de casser des habitudes de navigation.
- Conserver les noms de fichiers temporaires existants tant qu'un renommage n'a pas été validé.

## Markdown et Obsidian

- Ajouter un titre `#` en tête de note quand il manque.
- Commencer les notes structurées par un bloc `> [!summary]`.
- Utiliser des liens internes Obsidian `[[Nom de note]]`.
- Préférer des sections simples : `Quand utiliser`, `Prérequis`, `Commandes`, `Précautions`, `Notes associées`.

## Images

- Dossier d'images configuré : `z. Ressources/IMG`.
- Conserver le format d'intégration Obsidian : `![[Pasted image ...png]]`.
- Ne pas déplacer ni supprimer d'image sans validation explicite.
- Ajouter uniquement des légendes courtes quand elles clarifient la lecture.

## Git

- Travailler exclusivement sur la branche `refactor`.
- Vérifier avant toute session de modification :
  - `git status`
  - `git branch --show-current`
- Ne jamais modifier `main`.
- Ne jamais merger vers `main` sans validation manuelle dans Obsidian.
- Découper le travail en lots cohérents avec commits dédiés.

## Cadre éditorial

- Garder les notes centrées sur l'apprentissage, la certification, les labs et les environnements autorisés.
- Ajouter le rappel de cadre d'utilisation dans les notes sensibles.
- Ne pas enrichir les notes avec du contenu orienté usage réel non autorisé, destruction ou furtivité avancée.

