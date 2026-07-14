---
flag: 26
machine: dcorp-adminsrv
statut: à faire
technique: Constrained delegation (enum)
LO: [[LO16 - Constrained delegation]]
---

# F26 - Valeur msds-allowedtodelegateto adminsrv

> [!summary] Objectif
> Lire l'attribut `msDS-AllowedToDelegateTo` de `dcorp-adminsrv` — c'est la liste des SPN vers lesquels ce compte machine peut deleguer (donc les cibles S4U).

## Contexte
- **Etape attack flow**: prep constrained delegation abuse ([[LO16 - Constrained delegation]]) — on identifie d'abord la cible autorisee cote LDAP, ensuite on exploite via Rubeus s4u.
- **Ce qu on a**: shell studentx, PowerView charge, credentials domaine standard suffisent (attribut lisible en LDAP par tout user).
- **Ce qu on cherche**: la (ou les) valeur(s) SPN listee(s) dans `msDS-AllowedToDelegateTo` sur le compte machine `dcorp-adminsrv$`.

## Solution

### Etape 1: enumerer les comptes en constrained delegation
```powershell
. C:\AD\Tools\PowerView.ps1
Get-DomainComputer -TrustedToAuth | Select-Object samaccountname,msds-allowedtodelegateto
```
→ Voir [[Cheatsheet - Delegation attacks]].

### Etape 2: lire l'attribut precis sur adminsrv
```powershell
Get-DomainComputer -Identity dcorp-adminsrv -Properties samaccountname,msds-allowedtodelegateto
# ou en LDAP direct
Get-DomainComputer dcorp-adminsrv | Select-Object -ExpandProperty msds-allowedtodelegateto
```

### Etape 3 (optionnel): AD Module natif
```powershell
Get-ADComputer -Identity dcorp-adminsrv -Properties msDS-AllowedToDelegateTo
```

## Reponse

| Element | Valeur |
|---------|--------|
| msDS-AllowedToDelegateTo de dcorp-adminsrv | <A REMPLIR DEPUIS LAB> |

## Pieges et OPSEC
- L'attribut peut contenir plusieurs SPN — retenir la liste complete (souvent `time/dcorp-dc` ou similaire — cible S4U du flag suivant [[F27 - Service alternatif via constrained deleg]]).
- Lecture LDAP standard = tres OPSEC-safe (pas d'event 4662 particulier hors auditing SACL agressif).
- Ne pas confondre `msDS-AllowedToDelegateTo` (constrained classique, cote delegue) et `msDS-AllowedToActOnBehalfOfOtherIdentity` (RBCD, cote cible).

## Refs
- [[Cheatsheet - Delegation attacks]]
- [[Cheatsheet - RBCD]]
- [[LO16 - Constrained delegation]]
- [[F27 - Service alternatif via constrained deleg]]
- [[Synthèse - Kerberos]]
