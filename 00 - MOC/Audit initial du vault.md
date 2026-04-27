# Audit initial du vault

> [!summary] Résumé
> État initial du vault avant refactor : structure observée, points faibles de navigation et garde-fous à respecter sur `refactor`.

## Contexte Git

- Dépôt : `Robrtooo/CRTP-Notes`
- Branche de travail autorisée : `refactor`
- Branche à ne pas modifier : `main`

## Arborescence observée

- `1. Objectives/`
- `a. Active Directory/`
- `b. PowerShell/`
- `c. Offensiv .NET/`
- `Domain Enumeration/`
- `y. Tools/`
- `z. Ressources/`
- `.obsidian/`

## Configuration Obsidian

- Dossier d'images : `z. Ressources/IMG`
- Mise à jour automatique des liens : activée
- Format d'image dominant : `![[Pasted image ...png]]`

## Points de vigilance

- Noms de notes à corriger plus tard sans renommage immédiat :
  - `Domain Enumeration/CheetSheet.md`
  - `c. Offensiv .NET/e. Assume Breach/Sans titre.md`
  - `swdw.md`
- Liens mal formés repérés :
  - `[[[2. Good practices ]]`
- Certaines notes sensibles doivent être recadrées strictement pour un usage lab/certification.

## Contraintes de refactor

- Ne pas déplacer les images dans ce lot.
- Ne supprimer aucun fichier sans validation explicite.
- Ne supprimer aucune image sans validation explicite.
- Préserver les liens Obsidian existants autant que possible.

## Vérifications à poursuivre

- Vérifier les liens internes après refactor des notes prioritaires.
- Contrôler les images potentiellement orphelines avant toute action future.
- Identifier les notes à renommer plus tard après validation manuelle.
