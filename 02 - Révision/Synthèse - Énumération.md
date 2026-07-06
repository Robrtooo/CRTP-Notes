# Synthèse - Énumération

> [!summary] Résumé
> Ordre et outils d'une énumération AD. Cours : [[Énumération AD]]. Cartes : [[Flashcards - Énumération]].

## Pourquoi

Tout compte authentifié lit l'annuaire par LDAP → énumérer d'abord, largement, sans privilège.

## Ordre logique

1. Domaine, SID, DC, DNS.
2. Users : SPN (Kerberoast), preauth off (AS-REP), `adminCount=1`, descriptions.
3. Groupes (DA/EA, imbrications), ordinateurs (OS, délégations).
4. ACL (chemin vers DA), GPO/OU, trusts.
5. Sessions (chasse au DA), AD CS, MSSQL.
6. Graphe BloodHound → chemin le plus court vers Domain Admin.

## Outils selon la furtivité

- **PowerView / SharpView** : complet mais signé (→ lancer sous InviShell).
- **Module AD (ADWS)** : signé Microsoft, fonctionne en Constrained Language Mode.
- **BloodHound / SharpHound** : analyse en graphe. **SOAPHound** : collecte ADWS furtive.

## Réflexe

« Quel acquis, vers quelle attaque ? » — pas de collecte gratuite. Fractionner (jamais un SharpHound « All » d'un coup).

## Notes associées

- [[Énumération AD]]
- [[Cheatsheet - Reconnaissance AD]]
- [[Cheatsheet - BloodHound SharpHound]]
- [[Enumeration]]
