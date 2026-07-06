# Synthèse - Trusts et forêts

> [!summary] Résumé
> Franchir les frontières. Cours : [[Fondamentaux AD]]. Fiche : [[Cheatsheet - Trusts et forêts]].

## À retenir

- **Forêt = frontière de sécurité** ; l'objectif final est Enterprise Admin / la forêt.
- Un trust : lire ses attributs, ne pas supposer — **TrustType**, **TrustDirection**, **TrustAttributes**.
- **Intra-forêt (enfant → parent)** : SID History **non filtré** → injecter le SID des **Enterprise Admins** (519) dans un TGT inter-realm forgé → EA.
- **Inter-forêt** : le filtrage SID (`FILTER_SIDS`) limite la portée ; la **clé de trust** permet un referral ticket vers un service de l'autre forêt.

## Points de contrôle

Domaine source · domaine cible · TrustType · TrustDirection · TrustAttributes.

## Notes associées

- [[Cheatsheet - Trusts et forêts]] · [[Fondamentaux AD]] · [[Synthèse - Kerberos]] · [[2. Attack flow]]
