# Attaques AD

> [!summary] Objectif
> Comprendre le *mécanisme* de chaque grande famille d'attaque AD — pourquoi elle marche, pas seulement comment la lancer. Les cheatsheets donnent les commandes, [[2. Attack flow]] donne l'ordre d'enchaînement, ce cours donne la théorie sous-jacente. Révision : [[Flashcards - Attaques AD]].

## La logique commune

Presque toute attaque AD exploite l'un de ces quatre leviers : un **secret partagé** (Kerberos), une **DACL trop permissive**, une **délégation** mal cadrée, ou un **protocole de confiance** (réplication, trusts). Le fil rouge : transformer un acquis en un acquis supérieur, du simple compte jusqu'à la forêt.

## Attaques par secret Kerberos (offline)

- **Kerberoasting** — tout compte peut demander le TGS d'un compte de service à SPN ; ce TGS est chiffré avec le hash du compte de service, donc crackable offline. Marche parce que Kerberos *n'exige aucune autorisation* pour demander un ticket de service. Viser les comptes user (les comptes machine ont des mots de passe aléatoires). [[Cheatsheet - Kerberoasting]]
- **AS-REP Roasting** — si un compte a « pre-authentication » désactivée, le KDC renvoie une AS-REP partiellement chiffrée avec le hash du compte, sans qu'on prouve son identité → crack offline. [[Cheatsheet - AS-REP Roasting]]

> [!note] Pourquoi « offline »
> Dans les deux cas, on récupère un blob chiffré par le hash de la cible, puis on casse ce hash sur sa propre machine. Aucune tentative de connexion bruyante côté DC — d'où la furtivité relative.

## Attaques par forge de ticket (post-compromission)

- **Silver Ticket** — avec le hash d'un compte de service, on forge directement un TGS pour *son* service. On court-circuite le KDC : le service valide le ticket avec sa propre clé. Portée limitée à ce service, mais aucune interaction avec le DC.
- **Golden Ticket** — avec le hash de `krbtgt`, on forge un TGT arbitraire. On devient qui on veut, pour n'importe quel service, tant que krbtgt n'est pas changé deux fois. Persistance ultime.
- **Diamond Ticket** — variante furtive : au lieu de forger un TGT de zéro (Golden), on demande un vrai TGT au DC puis on le déchiffre/modifie avec la clé krbtgt (AES). Le PAC ressemble à un ticket légitime, ce qui déjoue les détections basées sur les anomalies de forge. Requiert la clé AES de krbtgt. [[Cheatsheet - Tickets Kerberos]]

## Attaques par DACL (contrôle d'objet)

Un droit d'écriture sur un objet devient un chemin d'attaque :

- **GenericAll / GenericWrite** sur un user → réinitialiser le mot de passe ou poser un SPN (targeted Kerberoasting).
- **WriteDACL** → s'octroyer soi-même des droits supplémentaires.
- **GenericWrite / GenericAll** sur un computer → **RBCD** (voir plus bas).
- **AdminSDHolder** → écrire dans cet objet modèle propage des droits vers tous les groupes protégés = persistance discrète.

Ces chemins se découvrent en masse avec BloodHound. [[Cheatsheet - ACL et droits AD]] · [[Cheatsheet - BloodHound SharpHound]]

## Attaques par délégation

La délégation Kerberos permet à un service d'agir *au nom* d'un utilisateur. Trois variantes, trois abus :

- **Unconstrained** — la machine stocke le TGT de tout utilisateur qui s'y authentifie. Si on la compromet, on peut forcer un DC à s'y connecter (coercition type Printer Bug) et capturer son TGT.
- **Constrained (S4U)** — le compte peut demander des tickets vers une liste de services précis au nom d'un utilisateur (`S4U2Self` + `S4U2Proxy`). Si on contrôle ce compte, on usurpe n'importe quel utilisateur *vers ces services*.
- **RBCD** — la cible déclare elle-même qui peut la déléguer, via `msDS-AllowedToActOnBehalfOf`. Un simple `GenericWrite` sur la cible suffit à s'y ajouter, puis à usurper Administrator face à cette machine. [[Cheatsheet - Delegation attacks]] · [[Cheatsheet - RBCD]]

## Attaques par protocole de confiance

- **DCSync** — on se fait passer pour un DC et on demande une réplication des secrets (`DRSGetNCChanges`). Le DC nous renvoie les hash, dont krbtgt. Nécessite les droits de réplication (Domain Admin, ou une DACL déléguée). Pas d'accès disque, pas de service à toucher. [[Cheatsheet - Persistence]]
- **Abus de trust** — la clé de trust inter-domaine permet de forger un referral ticket. Intra-forêt (enfant→parent), on y injecte le SID des Enterprise Admins (SID History non filtré) → EA. Inter-forêt, le filtrage SID limite la portée. [[Cheatsheet - Trusts et forêts]]

## Deux voies transverses

- **AD CS (ESC1/ESC3)** — un template de certificat mal configuré laisse un simple user demander un certificat *au nom d'un autre* (ex. un DA). Le certificat sert ensuite à obtenir un TGT. Court-circuite toute la chaîne classique. [[Cheatsheet - AD CS et certificats]]
- **MSSQL / database links** — une instance SQL qui fait confiance à un compte de domaine permet l'exécution de commandes, et les liens entre bases permettent de rebondir de serveur en serveur, y compris entre forêts. [[Cheatsheet - MSSQL et PowerUpSQL]]

## Notes associées
- [[2. Attack flow]] · [[Fondamentaux AD]] · [[Kerberos]] · [[Flashcards - Attaques AD]]
- [[Synthèse - Attaques AD]] · [[1. Cheatsheet - Toutes les commandes]]
