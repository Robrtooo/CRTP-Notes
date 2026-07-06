# Flashcards - Attack flow

> [!summary] Comment réviser
> Réponds de tête, puis déplie. Cours : [[2. Attack flow]].

## Principe

> [!question]- Quelle est l'idée centrale de l'attack flow ?
> L'attaque AD est une machine à états : on part de ce qu'on possède (un état) pour dériver les actions possibles, et chaque action réussie fait changer d'état. On enchaîne les transitions jusqu'à la domination de la forêt. Réflexe : « qu'est-ce que je possède, et vers quel acquis chaque option m'emmène ? »

## Progression d'état

> [!question]- Depuis un simple compte de domaine, quelles sont les 3 premières options à considérer ?
> Énumérer largement (AD est lisible par tout compte authentifié), Kerberoaster un compte à SPN, et lancer BloodHound pour trouver un chemin vers DA. En parallèle : repérer un droit ACL exploitable.

> [!question]- Tu as `GenericWrite` sur un objet computer — où ça mène ?
> Vers RBCD : écrire `msDS-AllowedToActOnBehalfOf` pour t'ajouter, puis S4U → accès admin sur cette machine en usurpant Administrator.

> [!question]- Tu es admin local sur une machine. Quelles transitions d'état viser ?
> Dumper les secrets en mémoire (hash/clés des comptes connectés), chasser une session de DA présente, et vérifier si la machine est en unconstrained delegation (capture de TGT + coercition du DC).

> [!question]- Tu obtiens un hash / une clé AES. Quelles options selon la nature du compte ?
> OverPass-the-Hash pour une session complète en tant que ce compte ; si c'est un compte de service/machine, Silver Ticket vers ses services ; si le compte est privilégié, tu passes à l'état DA.

> [!question]- Depuis Domain Admin, quelles actions de domination / persistance ?
> DCSync (extraire tout hash, dont krbtgt) → Golden/Diamond ticket ; DSRM (accès DC indépendant du domaine) ; extraction de la clé de trust pour franchir vers un autre domaine/forêt.

> [!question]- DA d'un domaine enfant : comment atteindre Enterprise Admin ?
> Forger un TGT inter-realm en injectant le SID des Enterprise Admins dans le SID History (trust enfant→parent intra-forêt non filtré) → EA.

## Voies transverses

> [!question]- Quels deux chemins court-circuitent souvent la progression classique, et pourquoi ?
> AD CS (ESC1/ESC3) : un simple Domain User → certificat au nom d'un DA → TGT DA/EA, sans passer par une compromission machine. MSSQL : un compte de domaine accepté par une instance SQL → database links → RCE cross-forest.

## Voir aussi

- [[2. Attack flow]]
- [[Flashcards - MOC]]
- [[Cours - MOC]]
