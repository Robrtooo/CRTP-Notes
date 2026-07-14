# Kerberos

> [!summary] Objectif
> Rappel du protocole Kerberos tel qu'il est utilisé dans AD, avec l'angle qui compte pour l'offensif : *quel secret chiffre quoi*. C'est cette mécanique qui rend possibles kerberoasting, AS-REP roasting et les tickets forgés. Révision : [[Flashcards - Kerberos]].

## Le modèle

Kerberos est un protocole d'authentification à tiers de confiance. Dans AD, ce tiers est le **KDC**, hébergé par chaque contrôleur de domaine. Le KDC a deux rôles :
- **AS** (Authentication Service) — délivre le TGT.
- **TGS** (Ticket Granting Service) — échange un TGT contre un ticket de service.

Deux catégories de secrets font tourner le tout :
- La clé du compte **`krbtgt`** chiffre et signe tous les **TGT** du domaine.
- La clé de chaque **compte de service** chiffre le **TGS** qui le concerne.

Retenir cette séparation : presque toutes les attaques Kerberos en découlent.

## Le flux d'authentification

1. **AS-REQ / AS-REP** — le client prouve son identité (préauthentification : un timestamp chiffré avec son hash) et reçoit un **TGT**, chiffré avec la clé de `krbtgt`. Le TGT est opaque pour le client, il ne fait que le rejouer.
2. **TGS-REQ / TGS-REP** — le client présente son TGT et demande l'accès à un service (identifié par son **SPN**). Le KDC renvoie un **TGS**, chiffré avec la clé du **compte de service** visé.
3. **AP-REQ** — le client présente le TGS directement au service, qui le déchiffre avec sa propre clé et lui accorde l'accès. Le KDC n'est plus dans la boucle.

> [!note] Le PAC
> Chaque ticket embarque un **PAC** (Privilege Attribute Certificate) : il porte le SID de l'utilisateur, ses groupes, etc. C'est lui qui détermine les privilèges effectifs. Le PAC est signé avec la clé de `krbtgt` — le forger ou le trafiquer, c'est s'attribuer les groupes de son choix.

## Pourquoi chaque secret crée une attaque

- **Préauth désactivée** → le KDC renvoie un AS-REP chiffré à quiconque le demande, sans vérifier l'identité → **AS-REP roasting**.
- **TGS chiffré avec la clé du compte de service** → n'importe quel user peut demander le TGS d'un compte à SPN et le cracker offline → **Kerberoasting**.
- **TGT chiffré avec la clé de `krbtgt`** → connaître cette clé permet de forger un TGT arbitraire → **Golden Ticket**.
- **TGS chiffré avec la clé du service** → connaître le hash du service permet de forger un TGS sans passer par le KDC → **Silver Ticket**.
- **PAC signé par `krbtgt`** → un vrai TGT dont on re-signe le PAC → **Diamond Ticket** (plus furtif que le Golden).

## RC4 vs AES

Le type de chiffrement (etype) d'un ticket compte pour l'OPSEC. Historiquement RC4 (`rc4_hmac`, etype 23) ; AES (`aes256`, etype 18) est le standard moderne. Demander ou forger un ticket en **RC4** alors que l'environnement est en AES constitue un *encryption downgrade*, signal classique de détection. D'où le réflexe `/aes256` et l'option `/rc4opsec` de Rubeus (ne roaster que les comptes qui ne supportent que RC4).

## Délégation Kerberos (survol)

La délégation permet à un service d'agir *au nom* d'un utilisateur vers un autre service. Trois variantes, détaillées dans [[Cheatsheet - Delegation attacks]] et [[Cheatsheet - RBCD]] :
- **Unconstrained** — le service reçoit le TGT complet de l'utilisateur (rejeu total possible).
- **Constrained** — via S4U2Self/S4U2Proxy, mais borné à une liste de SPN.
- **RBCD** — la cible déclare elle-même qui peut la déléguer (attribut `msDS-AllowedToActOnBehalfOf`).

## Notes associées
- [[Flashcards - Kerberos]]
- [[Cheatsheet - Tickets Kerberos]]
- [[Cheatsheet - Kerberoasting]]
- [[Cheatsheet - AS-REP Roasting]]
- [[Fondamentaux AD]]
- [[Synthèse - Kerberos]]
