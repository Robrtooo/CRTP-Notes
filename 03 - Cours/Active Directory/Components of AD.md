# Components of AD

> [!summary] Résumé
> Rappel des composants essentiels d'Active Directory et de la structure forêt/domaine à comprendre pour les labs CRTP.

## Composants clés

1. Schéma - Defines objects and their attributes
Ex : What could be attributes of a user objects ?

2. Query and index mechanism - Provides searching and publication of objects and their properties
-> Component that will be abuse

3. Global Catalog - Contains information about every object in the directory

4. Replication Service - Distributes information across domain controllers.

## Structure

![[Pasted image 20260211102326.png]]

*Légende : relation entre forêt, domaines et composants de l'annuaire.*

**Many of the attack tool for AD that exists, and that we will use works ONLY if ur present in the forest**

> [!warning] Point de vigilance
> Une compromission d'un domaine peut avoir des conséquences sur le reste de la forêt selon les relations de confiance et les privilèges présents. À réviser uniquement dans le cadre du lab.

## Notes associées

- [[General Notes]]
- [[Active Directory - MOC]]

