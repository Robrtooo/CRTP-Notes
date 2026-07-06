# Flashcards - Attaques AD

> [!summary] Révision active
> Cartes au format Spaced Repetition — déck `#flashcards/attaques-ad`. Ouvre la commande « Spaced Repetition: Review flashcards » pour réviser la file du jour. Cours : [[Attaques AD]].

#flashcards/attaques-ad

**Quels sont les quatre leviers exploités par presque toute attaque AD ?**
?
Un secret partagé (Kerberos), une DACL trop permissive, une délégation mal cadrée, ou un protocole de confiance (réplication, trusts). Fil rouge : transformer un acquis en un acquis supérieur.

**Pourquoi le Kerberoasting fonctionne-t-il sans privilège ?**
?
Kerberos n'exige aucune autorisation pour demander le TGS d'un service. Ce TGS est chiffré avec le hash du compte de service, donc crackable offline. On vise les comptes user (les comptes machine ont des mots de passe aléatoires incassables).

**Différence de mécanisme entre AS-REP Roasting et Kerberoasting ?**
?
AS-REP roasting cible un compte dont la pré-authentification est désactivée : le KDC renvoie une AS-REP chiffrée avec le hash du compte sans preuve d'identité. Kerberoasting cible un compte à SPN et récupère un TGS. Les deux se cassent offline.

**Silver vs Golden vs Diamond ticket : mécanisme et secret requis ?**
?
Silver : hash d'un compte de service → TGS forgé pour son service, sans DC (portée limitée). Golden : hash de krbtgt → TGT arbitraire (persistance totale). Diamond : demande un vrai TGT puis le modifie avec la clé AES de krbtgt → PAC légitime, plus furtif que Golden.

**Comment un simple `GenericWrite` sur un objet computer mène-t-il à un accès admin ?**
?
Via RBCD : on écrit `msDS-AllowedToActOnBehalfOf` pour s'ajouter comme délégué autorisé, puis S4U pour usurper Administrator face à cette machine.

**Décris les trois variantes de délégation et leur abus.**
?
Unconstrained : la machine stocke les TGT ; on la compromet + coercition d'un DC → capture du TGT du DC. Constrained (S4U) : le compte usurpe un user vers une liste de services précis. RBCD : la cible déclare qui la délègue ; un GenericWrite suffit à s'ajouter.

**Comment fonctionne DCSync et que nécessite-t-il ?**
?
On se fait passer pour un DC et on demande une réplication des secrets (DRSGetNCChanges) ; le DC renvoie les hash, dont krbtgt. Nécessite les droits de réplication (Domain Admin ou DACL déléguée). Aucun accès disque au DC.

**Pourquoi AD CS (ESC1/ESC3) court-circuite-t-il la chaîne classique ?**
?
Un template de certificat mal configuré laisse un simple Domain User demander un certificat au nom d'un autre compte (ex. un DA). Le certificat sert ensuite à obtenir un TGT DA/EA, sans compromission machine intermédiaire.

## Voir aussi
[[Attaques AD]] · [[Cours - MOC]]
