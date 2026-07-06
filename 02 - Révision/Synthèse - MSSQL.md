# Synthèse - MSSQL

> [!summary] Résumé
> SQL Server comme voie transverse. Fiche : [[Cheatsheet - MSSQL et PowerUpSQL]]. Chemin : [[2. Attack flow]].

## À retenir

- Un **compte de domaine** peut être accepté par une instance SQL sans être administrateur du domaine.
- **PowerUpSQL** : découverte d'instances (`Get-SQLInstanceDomain`), infos serveur, liens.
- **Database links** : une instance fait confiance à une autre → **crawl** de proche en proche, y compris **cross-forest**.
- Exécution : **`xp_cmdshell`** (si activable) via `Get-SQLServerLinkCrawl -Query "..."`.

## Distinguer les trois étapes

Découverte d'instance → découverte/crawl des liens → exécution de requête/commande.

## Réflexe

Cartographier les liens avant d'exécuter ; un lien peut mener hors de la forêt de départ.

## Notes associées

- [[Cheatsheet - MSSQL et PowerUpSQL]]
- [[Attaques AD]]
- [[2. Attack flow]]
