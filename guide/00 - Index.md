# Guide — Fiches de dépannage CRTP

Fiches issues des sessions de debug. Chaque fiche = un problème rencontré + cause + fix.

## Fiches
- [[01 - Charger PowerView]] — `Get-DomainSID: not recognized`
- [[02 - Enterprise Admins & referral]] — `A referral was returned from the server`
- [[03 - Enumeration ACLs - sens de la requete]] — qui a du pouvoir sur quoi ?
- [[04 - Debug des requetes ACL]] — pourquoi mon filtre renvoie toujours vide
- [[05 - Lancer PowerShell en bypass]] — `powershell -nop -ep bypass` + flags
- [[06 - Methode - utiliser un script ps1]] — recette dot-source générique (PowerView & co)

## Réflexe général
> [!TIP] Avant de filtrer, INSPECTE
> Ne filtre jamais à l'aveugle. Sors **une** ligne brute complète avec `| fl *` ou `| Get-Member`, repère les **vrais noms de champs** et leurs **valeurs** (nom ou SID ?), PUIS construis le filtre.

> [!tip] Et après l'énumération ?
> Ces fiches couvrent seulement l'énum + le chargement d'outils. La suite du lab (BloodHound, privesc, lateral, escalade domaine) vit dans l'autre arbre : [[2. Attack flow]] · [[1. CRTP - Index]]. *Sans ce pont, on croit avoir fini alors qu'on est bloqué à l'énum.*
