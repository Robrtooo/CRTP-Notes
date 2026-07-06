# Synthèse - Attaques AD

> [!summary] Résumé
> Familles d'attaques et leur logique. Cours : [[Attaques AD]]. Chemin : [[2. Attack flow]]. Cartes : [[Flashcards - Attaques AD]].

## Quatre leviers

Secret Kerberos · DACL trop large · délégation mal cadrée · protocole de confiance (réplication, trust).

## Par famille (prérequis → gain)

- **Kerberoasting** : compte + SPN cible → hash de service crackable offline.
- **AS-REP Roasting** : compte sans pré-authentification → hash crackable offline.
- **ACL** : GenericAll/GenericWrite, WriteDACL, ForceChangePassword → contrôle d'objet.
- **RBCD** : GenericWrite sur un computer → admin via S4U.
- **Délégation** : unconstrained (+ coercition d'un DC) / constrained (S4U).
- **Tickets forgés** : Silver (clé service), Golden (krbtgt), Diamond (vrai TGT re-signé, furtif).
- **DCSync** : droits de réplication → tous les hash (dont krbtgt).
- **Trust** : clé de trust + SID History → Enterprise Admin / autre forêt.
- **Transverses** : AD CS (ESC1/ESC3), MSSQL database links.

## Réflexe

Prérequis exact → information recherchée → gain → état suivant. Valider avec les sorties du lab, pas les suppositions.

## Notes associées

- [[Attaques AD]]
- [[2. Attack flow]]
- [[Attaques AD - MOC]]
- [[1. Cheatsheet - Toutes les commandes]]
