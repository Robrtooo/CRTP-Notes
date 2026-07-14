# Synthèse - ACL et GPO

> [!summary] Résumé
> Contrôle d'objet par les droits. Cours lié : [[Attaques AD]]. Cartes : [[Flashcards - Attaques AD]].

## À retenir

- La **DACL** dit qui contrôle un objet et comment ; c'est un chemin d'attaque à part entière.
- Droits clés : **GenericAll**, **GenericWrite**, **WriteDACL**, **WriteOwner**, **ForceChangePassword**, **AllExtendedRights**.
- Une **GPO** est un objet AD → elle a sa propre DACL. Droit d'écriture sur une GPO liée à une OU = exécution de code sur toutes les machines de l'OU.
- **AdminSDHolder** : écrire dedans propage des droits vers les groupes protégés (persistance discrète).

## Enchaînements types

- GenericWrite sur un user → poser un SPN (targeted Kerberoast) ou reset du mot de passe.
- GenericWrite/GenericAll sur un computer → **RBCD**.
- WriteDACL → s'octroyer soi-même GenericAll.

## Réflexe

Objet précis → droit exact → impact concret. BloodHound révèle ces chemins en masse.

## Notes associées

- [[Cheatsheet - ACL et droits AD]]
- [[Cheatsheet - GPO et OU]]
- [[Cheatsheet - RBCD]]
- [[Attaques AD]]
