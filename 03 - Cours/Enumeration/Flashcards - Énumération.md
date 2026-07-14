# Flashcards - Énumération

> [!summary] Comment réviser
> Réponds de tête, puis déplie. Cours : [[Énumération AD]].

> [!question]- Pourquoi peut-on énumérer AD sans aucun privilège ?
> AD est conçu pour être interrogeable : tout compte authentifié peut lire par LDAP l'essentiel de l'annuaire (users, groupes, ordinateurs, trusts, la plupart des ACL). Ce n'est pas une faille, c'est le fonctionnement normal du service.

> [!question]- Quels attributs utilisateurs cherche-t-on en priorité, et pour quoi ?
> Comptes à SPN (Kerberoasting), sans pré-authentification (AS-REP roasting), `adminCount=1` (comptes protégés/privilégiés), et les descriptions (mots de passe laissés en clair).

> [!question]- Pourquoi énumérer les ACL est-il souvent décisif ?
> Parce que les droits inter-objets (GenericAll, WriteDACL, ForceChangePassword…) forment fréquemment un chemin reliant un compte quelconque à Domain Admin, sans aucune faille logicielle.

> [!question]- Quel est l'intérêt du module Active Directory par rapport à PowerView ?
> Il utilise des cmdlets Microsoft légitimes via ADWS, signées Microsoft, qui fonctionnent même en Constrained Language Mode : utile quand PowerView est bloqué ou trop surveillé.

> [!question]- Qu'apporte BloodHound que les requêtes manuelles n'apportent pas ?
> L'analyse en graphe : à partir de milliers de faits collectés, il calcule automatiquement le chemin le plus court vers Domain Admin. Il transforme la donnée brute en décision.

> [!question]- Pourquoi préférer SOAPHound dans un environnement surveillé ?
> Il collecte via ADWS (SOAP) au lieu de LDAP direct, ce qui est plus furtif face aux détections qui surveillent les requêtes LDAP massives.

> [!question]- Quelle question doit guider chaque étape d'énumération ?
> « Quel acquis cet élément me donne-t-il, et vers quelle attaque il pointe ? » On n'énumère pas pour collecter, mais pour alimenter le chemin d'attaque.

> [!question]- Deux réflexes OPSEC lors de l'énumération ?
> Fractionner les collectes (éviter un SharpHound « All » d'un coup, très bruyant) et lancer PowerView sous InviShell / privilégier le module AD signé quand la surveillance est forte.

## Voir aussi
- [[Énumération AD]]
