# Synthèse - Kerberos

> [!summary] Résumé
> Rappel express du protocole et des attaques dérivées. Cours : [[Kerberos]]. Cartes : [[Flashcards - Kerberos]].

## Le flux

- **AS-REQ / AS-REP** : pré-authentification = timestamp chiffré avec le hash du user → **TGT** (chiffré par **krbtgt**).
- **TGS-REQ / TGS-REP** : TGT + SPN → **TGS** (chiffré par la clé du **compte de service**).
- **AP-REQ** : TGS présenté au service, qui le déchiffre avec sa propre clé — le KDC n'est plus dans la boucle.
- **PAC** (droits de l'utilisateur) signé par krbtgt, embarqué dans le ticket.

## Chaque secret = une attaque

- Pré-auth désactivée → **AS-REP Roasting**.
- TGS d'un compte à SPN → **Kerberoasting**.
- Clé de **krbtgt** → **Golden Ticket** (TGT arbitraire).
- Clé d'un service → **Silver Ticket** (TGS forgé, sans passer par le KDC).
- Vrai TGT re-signé avec la clé AES de krbtgt → **Diamond Ticket** (furtif).

## RC4 vs AES

Forcer RC4 sur un compte AES-only = downgrade visible (event 4769 avec type de chiffrement anormal). Préférer AES ; `/rc4opsec` côté Rubeus.

## Notes associées

- [[Kerberos]]
- [[Cheatsheet - Tickets Kerberos]]
- [[Cheatsheet - Kerberoasting]]
- [[Cheatsheet - AS-REP Roasting]]
