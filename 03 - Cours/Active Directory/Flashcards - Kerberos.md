# Flashcards - Kerberos

> [!summary] Comment réviser
> Réponds de tête, puis déplie. Cours : [[Kerberos]].

> [!question]- Quels sont les deux rôles du KDC ?
> AS (Authentication Service) qui délivre le TGT, et TGS (Ticket Granting Service) qui échange un TGT contre un ticket de service. Le KDC est hébergé par le contrôleur de domaine.

> [!question]- Avec quelle clé un TGT est-il chiffré ? Et un TGS ?
> Le TGT est chiffré avec la clé de `krbtgt`. Le TGS est chiffré avec la clé du **compte de service** visé. Cette distinction est la racine de la plupart des attaques Kerberos.

> [!question]- Décris le flux d'authentification en 3 échanges.
> 1) AS-REQ/AS-REP : préauth (timestamp chiffré avec le hash du user) → TGT. 2) TGS-REQ/TGS-REP : on présente le TGT + un SPN → TGS. 3) AP-REQ : on présente le TGS au service, qui le déchiffre avec sa clé — le KDC n'est plus dans la boucle.

> [!question]- Qu'est-ce que le PAC et pourquoi est-il critique côté offensif ?
> Le Privilege Attribute Certificate est embarqué dans le ticket et porte le SID + les groupes de l'utilisateur (donc ses privilèges effectifs). Il est signé avec la clé de `krbtgt` : forger/trafiquer le PAC = s'attribuer les groupes voulus (base du Golden et du Diamond ticket).

> [!question]- Pourquoi le Kerberoasting fonctionne-t-il sans aucun privilège ?
> Parce que n'importe quel user du domaine peut demander le TGS d'un compte à SPN, et que ce TGS est chiffré avec la clé du compte de service — donc crackable offline. Aucune élévation nécessaire pour la demande.

> [!question]- Qu'est-ce qui rend un compte vulnérable à l'AS-REP roasting ?
> La préauthentification désactivée (`DoesNotRequirePreAuth`) : le KDC renvoie alors l'AS-REP chiffré à quiconque le demande, sans vérifier l'identité au préalable — d'où un hash crackable offline.

> [!question]- Golden vs Silver ticket : quel secret, quelle portée ?
> Golden : clé de `krbtgt` → TGT forgé → tout le domaine. Silver : clé d'un compte de service → TGS forgé → un seul service, sans jamais contacter le KDC (donc plus discret).

> [!question]- Qu'apporte le Diamond ticket par rapport au Golden ?
> Même prérequis (clé `krbtgt`) mais on part d'un vrai TGT dont on re-signe le PAC, au lieu de forger de zéro. Résultat identique (impersonation totale) mais plus furtif face aux détections qui repèrent un ticket 100 % fabriqué.

> [!question]- Pourquoi préférer AES à RC4 lors d'une attaque Kerberos ?
> Demander/forger un ticket en RC4 dans un environnement AES est un *encryption downgrade*, un signal de détection classique. D'où `/aes256` et l'option `/rc4opsec` de Rubeus.

> [!question]- Les 3 types de délégation Kerberos, en une phrase chacun ?
> Unconstrained : le service capte le TGT complet de l'utilisateur. Constrained : S4U vers une liste de SPN autorisés. RBCD : la cible déclare elle-même qui peut la déléguer (`msDS-AllowedToActOnBehalfOf`).

## Voir aussi
- [[Kerberos]]
