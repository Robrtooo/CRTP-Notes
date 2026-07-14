# Synthèse - Active Directory

> [!summary] Résumé
> Rappel express des fondamentaux AD. Cours : [[Fondamentaux AD]]. Cartes : [[Flashcards - Fondamentaux AD]].

## Structure

- **Forêt** = frontière de sécurité (objectif final : Enterprise Admin). **Domaine** = frontière d'administration/réplication. **OU** = délégation + application de GPO.
- Les domaines d'une forêt se font confiance automatiquement : trusts parent-enfant bidirectionnels et transitifs.

## Cœur technique

- **DC** héberge `NTDS.dit` (tous les hash du domaine) + rôle **KDC**. Pivot = **krbtgt** (chiffre tous les TGT).
- **Tout est objet** + **DACL** → une grande partie des attaques exploite une ACL trop permissive, pas une faille logicielle.

## À mémoriser

- SID `S-1-5-21-<domaine>-<RID>`. RID : **512** Domain Admins, **519** Enterprise Admins, **518** Schema Admins, **513** Domain Users, **500** Administrator, **502** krbtgt.
- Composants abusés : **query/index LDAP** (énumération sans privilège), **réplication** (→ DCSync), **catalogue global** (recherche à l'échelle de la forêt).

## Réflexe

Relier chaque objet/attribut à l'attaque qu'il ouvre : SPN → Kerberoast, preauth off → AS-REP, GenericWrite → RBCD, droits de réplication → DCSync.

## Notes associées

- [[Fondamentaux AD]]
- [[Kerberos]]
- [[Attaques AD]]
- [[Attack flow]]
- [[Fondamentaux AD]]
- [[Fondamentaux AD]]
