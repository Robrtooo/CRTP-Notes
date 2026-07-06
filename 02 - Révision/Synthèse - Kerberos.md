# Synthèse - Kerberos

> [!summary] Résumé
> Rappel rapide des principes Kerberos utiles aux labs CRTP : comptes, tickets, SPN et préauthentification.

> [!warning] Cadre d'utilisation
> À utiliser uniquement dans un environnement de lab autorisé, dans le cadre de l'apprentissage et de la préparation CRTP.

## À retenir

- Kerberos repose sur des échanges de tickets plutôt que sur des mots de passe transmis directement au service.
- Les SPN relient des services à des comptes, ce qui explique leur rôle dans Kerberoasting.
- La préauthentification protège les comptes contre certaines demandes triviales d'AS-REP.

## Tickets à connaître

- TGT : ticket initial délivré par le KDC.
- TGS : ticket de service demandé pour accéder à un service particulier.

## Points de vigilance

- Toujours relier le ticket observé au compte et au service concernés.
- Vérifier si le scénario porte sur un compte de service, un compte sans préauthentification ou une délégation.

## Notes associées

- [[Cheatsheet - Kerberoasting]]
- [[Cheatsheet - AS-REP Roasting]]
- [[Cheatsheet - Delegation attacks]]
- [[Components of AD]]

