# Énumération AD

> [!summary] Objectif
> Comprendre pourquoi l'énumération est possible sans privilège, quoi cartographier, et avec quel outil selon la contrainte de furtivité. C'est l'étape qui transforme un accès aveugle en carte d'attaque. Pointeurs d'outils : [[Enumeration]], [[CheetSheet]]. Commandes : [[Cheatsheet - Reconnaissance AD]]. Révision : [[Flashcards - Énumération]].

## Pourquoi ça marche sans privilège

Active Directory est conçu pour être *interrogeable* : tout compte authentifié peut lire par LDAP l'essentiel de l'annuaire — utilisateurs, groupes, ordinateurs, relations d'approbation, la plupart des ACL. Il n'y a rien à « exploiter » pour énumérer ; c'est le fonctionnement normal du service. D'où la règle : depuis un simple compte de domaine, **on énumère d'abord, largement, avant d'agir**.

## Quoi cartographier (et pourquoi)

- **Domaine, SID, DC** — le socle : périmètre, identifiant de domaine (pour la forge de tickets), où adresser les requêtes.
- **Utilisateurs** — repérer les comptes à SPN (Kerberoasting), sans pré-auth (AS-REP), à `adminCount=1` (comptes protégés/privilégiés), et lire les descriptions (mots de passe oubliés en clair).
- **Groupes** — qui est Domain/Enterprise Admin, imbrications de groupes, appartenances non évidentes.
- **Ordinateurs** — versions d'OS, serveurs à haute valeur, machines en délégation.
- **ACL** — les droits inter-objets : c'est là que se cache le chemin d'un compte quelconque vers DA.
- **GPO et OU** — qui peut modifier quoi, quelles machines sont ciblées.
- **Trusts** — les autres domaines/forêts accessibles, direction et transitivité.
- **Sessions** — *où* les comptes privilégiés sont connectés (chasse au DA).
- **Délégations, AD CS, MSSQL** — les voies rapides (unconstrained/constrained/RBCD, templates ESC, instances SQL).

> [!note] La question qui guide l'énumération
> Ne pas collecter pour collecter. À chaque objet intéressant, se demander : « quel acquis cet élément me donne-t-il, et vers quelle attaque il pointe ? » L'énumération alimente directement [[Attack flow]].

## Avec quoi (choix selon la furtivité)

- **PowerView / SharpView** — la boîte à outils de référence. Très complète (ACL, trusts, sessions, GPO), mais très signée : à utiliser avec [[Cheatsheet - PowerShell|InviShell]] pour contourner le logging.
- **Module Active Directory (`ActiveDirectory`)** — cmdlets Microsoft légitimes, passant par ADWS. Signé Microsoft et fonctionnel même en Constrained Language Mode : précieux quand PowerView est bloqué.
- **BloodHound / SharpHound** — collecte massive puis analyse *en graphe* : trouve automatiquement le chemin le plus court vers Domain Admin. L'outil qui transforme des milliers de faits en une décision. [[Cheatsheet - BloodHound SharpHound]]
- **SOAPHound** — collecteur passant par ADWS (SOAP) plutôt que LDAP direct : plus furtif face aux détections qui surveillent les requêtes LDAP massives.

## OPSEC de l'énumération

Une requête LDAP isolée ressemble à du trafic normal ; une collecte SharpHound « All » d'un coup est bruyante. Fractionner, préférer le module AD signé quand la surveillance est forte, et lancer PowerView sous InviShell. La reconnaissance discrète conditionne toute la suite. [[Cheatsheet - Monitoring et détections]]

## Notes associées
- [[Fondamentaux AD]] · [[Attack flow]] · [[Flashcards - Énumération]]
- [[Synthèse - Énumération]] · [[Cheatsheet - LDAP SMB Kerberos]]
