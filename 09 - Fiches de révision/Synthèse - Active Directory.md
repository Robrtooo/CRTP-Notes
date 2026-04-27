# Synthèse - Active Directory

> [!summary] Résumé
> Vue d'ensemble d'Active Directory pour réviser les concepts, composants et relations utiles à la préparation CRTP.

> [!warning] Cadre d'utilisation
> À utiliser uniquement dans un environnement de lab autorisé, dans le cadre de l'apprentissage et de la préparation CRTP.

## À retenir

- Active Directory centralise l'identité, l'authentification et l'autorisation dans un environnement Windows.
- Les objets AD portent des attributs qui deviennent des sources majeures d'énumération.
- Les relations entre utilisateurs, groupes, ordinateurs, OU, domaines et forêts comptent autant que les objets eux-mêmes.

## Composants clés

- Schéma : définit les objets et leurs attributs.
- LDAP : sert à interroger et publier les informations de l'annuaire.
- Kerberos : gère l'authentification principale en domaine.
- SMB : expose des ressources et sert souvent de support à l'administration.
- Global Catalog : agrège des informations utiles à l'échelle de la forêt.

## Questions de révision

- Quels objets sont les plus intéressants pour une première énumération ?
- Quelles différences pratiques entre domaine, forêt et OU ?
- Quels attributs reviennent souvent dans les chemins d'attaque ?

## Notes associées

- [[General Notes]]
- [[Components of AD]]
- [[Enumeration]]
- [[Active Directory - MOC]]

