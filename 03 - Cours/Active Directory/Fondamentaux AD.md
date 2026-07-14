# Fondamentaux AD

> [!summary] Objectif
> Comprendre la structure d'Active Directory du point de vue de l'attaquant : quels sont les objets, où vit le secret, et pourquoi la forêt — et non le domaine — est la vraie frontière de sécurité. Ce cours pose le vocabulaire mobilisé par toutes les cheatsheets. Révision : [[Flashcards - Fondamentaux AD]].

## Ce qu'est réellement Active Directory

Un annuaire : une base de données hiérarchique qui centralise l'authentification et l'autorisation d'un réseau Windows. Le principe fondateur : **tout est un objet** (utilisateurs, machines, groupes, GPO, unités d'organisation). Un objet porte des attributs, et surtout une liste de contrôle d'accès (DACL) qui dit *qui a le droit de faire quoi dessus*. Cette dernière propriété est la source de la moitié des attaques modernes : compromettre AD, c'est souvent exploiter une DACL mal configurée, pas une faille logicielle.

> [!NOTE] Définition Microsoft
> Active Directory enables centralized, secure management of an entire network, which might span a building, a city or multiple locations throughout the world.

En pratique, la sécurité réelle dépend fortement de la maintenance, des durcissements et de l'hygiène d'administration — la centralisation n'est un gain défensif que si l'annuaire est correctement tenu.

![[Pasted image 20260211101806.png]]

*Légende : vue schématique générale d'un environnement Active Directory.*

## La hiérarchie : OU, domaine, arbre, forêt

- **Unité d'organisation (OU)** — conteneur logique interne à un domaine. Sert à déléguer l'administration et à appliquer des GPO à un sous-ensemble d'objets.
- **Domaine** — frontière de réplication et d'administration. Partage une base d'annuaire, une politique de mot de passe, un espace de noms. Les Domain Admins y règnent.
- **Arbre** — un ou plusieurs domaines partageant un espace de noms contigu (`corp.local`, `eu.corp.local`).
- **Forêt** — l'ensemble des arbres partageant un même schéma et un catalogue global. Les domaines d'une forêt se font *automatiquement* confiance (trusts parent-enfant bidirectionnels et transitifs).

> [!important] La forêt est la frontière de sécurité, pas le domaine
> Contrairement à l'intuition, compromettre un domaine ne « contient » pas la brèche. À l'intérieur d'une forêt, les relations de confiance et les privilèges *Enterprise* permettent de rebondir d'un domaine à l'autre. L'objectif final d'un pentest AD n'est donc pas Domain Admin mais **Enterprise Admin** / la compromission de la forêt. C'est aussi pourquoi beaucoup d'outils d'attaque ne fonctionnent que si l'on opère depuis une machine *membre* de la forêt.

![[Pasted image 20260211102326.png]]

*Légende : relation entre forêt, domaines et composants de l'annuaire.*

> [!warning] Point de vigilance
> Une compromission d'un domaine peut avoir des conséquences sur le reste de la forêt selon les relations de confiance et les privilèges présents. À réviser uniquement dans le cadre du lab.

## Le contrôleur de domaine et le secret

Le **contrôleur de domaine (DC)** héberge la base d'annuaire `NTDS.dit`, qui contient *tous* les hash de mots de passe du domaine. Le DC joue aussi le rôle de **KDC** (centre de distribution de clés Kerberos). Le compte pivot est **`krbtgt`** : son hash chiffre tous les TGT du domaine — le voler, c'est pouvoir forger n'importe quelle identité (Golden Ticket). Voir [[Kerberos]].

## Les composants internes (et lesquels s'abusent)

- **Schéma** — définit les classes d'objets et leurs attributs possibles. Partagé par toute la forêt.
- **Mécanisme de requête et d'index** — permet la recherche LDAP. C'est *le* composant qu'on abuse : tout compte authentifié peut lire une immense partie de l'annuaire.
- **Catalogue global (GC)** — index partiel de *tous* les objets de la forêt, hébergé sur certains DC. Permet les recherches inter-domaines.
- **Service de réplication** — synchronise l'annuaire entre DC. Le protocole de réplication est précisément ce que **DCSync** détourne pour extraire des hash sans toucher au disque du DC.

## Identités : SID, RID, groupes

Chaque objet de sécurité possède un **SID** (identifiant de sécurité) de la forme `S-1-5-21-<domaine>-<RID>`. Le **RID** final identifie l'objet dans son domaine. Certains RID sont universels (« well-known ») :

- `512` Domain Admins · `519` Enterprise Admins · `518` Schema Admins
- `513` Domain Users · `500` compte Administrator du domaine · `502` krbtgt

Connaître ces RID est essentiel pour la forge de tickets et l'injection de **SID History** (par exemple injecter le SID `...-519` pour usurper les Enterprise Admins).

Les **groupes** ont une portée (global, domain local, universal) qui détermine où ils peuvent être utilisés. Les groupes à surveiller : Domain Admins, Enterprise Admins, Administrators, Account Operators, Backup Operators, DNSAdmins — chacun offre un chemin d'élévation direct ou indirect.

## GPO et trusts en une phrase chacun

- **GPO (Group Policy Object)** — objet de configuration/sécurité lié à une OU, un site ou un domaine ; s'applique à toutes les machines/utilisateurs du périmètre lié. Un droit d'écriture sur une GPO = exécution de code sur toutes les machines concernées ([[Cheatsheet - GPO et OU]]).
- **Trust (relation d'approbation)** — lien qui permet à un domaine d'accepter les authentifications d'un autre. Caractérisé par sa direction, sa transitivité et son périmètre (intra- ou inter-forêt). C'est l'autoroute vers Enterprise Admin ou vers une autre forêt ([[Cheatsheet - Trusts et forêts]]).

## Pourquoi c'est exploitable

Trois faits structurels rendent AD attaquable même sans faille :

1. **Lisibilité** — tout compte authentifié peut énumérer l'essentiel de l'annuaire par LDAP. La reconnaissance ne demande aucun privilège.
2. **DACL** — les droits sur les objets sont souvent trop larges (délégations oubliées, GenericAll accordé par confort). Un chemin d'ACL relie fréquemment un compte quelconque à Domain Admin.
3. **Kerberos par conception** — chaque secret partagé (compte de service, krbtgt, clé de trust) crée une attaque possible dès qu'il fuit. Voir [[Kerberos]].

## Notes associées
- [[Attaques AD]]
- [[Enumeration]]
- [[2. Attack flow]]
- [[Flashcards - Fondamentaux AD]]
- [[Synthèse - Active Directory]]
- [[Active Directory - MOC]]
