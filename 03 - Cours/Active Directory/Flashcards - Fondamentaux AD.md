# Flashcards - Fondamentaux AD

> [!summary] Comment réviser
> Réponds de tête, puis déplie. Cours : [[Fondamentaux AD]].

> [!question]- Pourquoi dit-on que « tout est un objet » dans AD, et pourquoi est-ce important pour l'attaquant ?
> Utilisateurs, machines, groupes, GPO, OU sont tous des objets portant des attributs et une DACL (qui peut faire quoi dessus). Compromettre AD revient souvent à exploiter une DACL trop permissive, pas une faille logicielle.

> [!question]- Quelle est la vraie frontière de sécurité : le domaine ou la forêt ? Conséquence ?
> La forêt. Les domaines d'une même forêt se font automatiquement confiance (trusts parent-enfant bidirectionnels transitifs), donc compromettre un domaine permet de rebondir. L'objectif final n'est pas Domain Admin mais Enterprise Admin / la forêt.

> [!question]- Qu'héberge le contrôleur de domaine, et quel est le compte pivot ?
> Il héberge `NTDS.dit` (tous les hash du domaine) et joue le rôle de KDC. Le compte pivot est `krbtgt` : son hash chiffre tous les TGT ; le voler permet de forger n'importe quelle identité (Golden Ticket).

> [!question]- Quels composants internes d'AD sont directement détournés en attaque ?
> Le mécanisme de requête/index (LDAP lisible par tout compte authentifié → énumération) et le service de réplication (détourné par DCSync pour extraire des hash sans toucher au disque du DC).

> [!question]- À quoi sert le catalogue global ?
> C'est un index partiel de tous les objets de la forêt, hébergé sur certains DC, qui permet les recherches inter-domaines.

> [!question]- Donne les RID « well-known » à connaître.
> 512 Domain Admins, 519 Enterprise Admins, 518 Schema Admins, 513 Domain Users, 500 Administrator, 502 krbtgt. Utiles pour la forge de tickets et l'injection de SID History (ex. injecter …-519 pour usurper les Enterprise Admins).

> [!question]- Qu'est-ce qu'une GPO et pourquoi un droit d'écriture dessus est dangereux ?
> Objet de configuration/sécurité lié à une OU, un site ou un domaine, appliqué à toutes les machines/utilisateurs du périmètre. Un droit d'écriture = exécution de code sur toutes les machines concernées.

> [!question]- Cite les trois faits structurels qui rendent AD attaquable sans faille logicielle.
> 1) Lisibilité : tout compte authentifié énumère l'annuaire par LDAP sans privilège. 2) DACL trop larges reliant un compte quelconque à Domain Admin. 3) Kerberos par conception : chaque secret partagé qui fuit crée une attaque.

## Voir aussi
- [[Fondamentaux AD]]
- [[Cours - MOC]]
