---
flag: 36
machine: dcorp-dc
statut: à faire
technique: ADCS-enroll-rights
LO: [[LO21 - AD CS (ESC1 et ESC3)]]
---

# F36 · Groupe droits enroll sur CA-Integration

> [!summary] Objectif
> Identifier le **groupe** listé dans `Enrollment Rights` du template `CA-Integration` — condition d'accès pour tenter un abus AD CS via ce template.

## Contexte
- **Etape attack flow**: cartographie fine des templates AD CS. Savoir *qui* a le droit d'enroll conditionne si le compte courant peut abuser le template ou s'il faut d'abord pivoter.
- **Ce qu on a**: session Domain User + énumération Certify.
- **Ce qu on cherche**: le **nom exact du groupe** (`<DOMAIN>\<GroupName>`) dans la section `Enrollment Rights` du template `CA-Integration`.

## Solution

### Etape 1: Enumeration ciblee
```powershell
C:\AD\Tools\Certify.exe find
```
Dans la longue sortie, dérouler jusqu'au bloc `CA Name` / `Template Name : CA-Integration`. Lire la section `Enrollment Rights`.

### Etape 2: Filtrage sortie (rapide)
```powershell
C:\AD\Tools\Certify.exe find | Select-String -Pattern "CA-Integration" -Context 0,25
```
Le champ `Enrollment Rights` liste les principaux autorisés (généralement 1-2 groupes + les admins par défaut).

### Etape 3 (alternative — LDAP direct)
```powershell
# via PowerView si Certify indisponible
Get-DomainObjectAcl -SearchBase "CN=CA-Integration,CN=Certificate Templates,CN=Public Key Services,CN=Services,CN=Configuration,DC=dollarcorp,DC=moneycorp,DC=local" -ResolveGUIDs | ? {$_.ObjectAceType -match "Enroll"}
```

## Reponse

| Element | Valeur |
|---------|--------|
| Groupe avec Enroll sur `CA-Integration` | <A REMPLIR DEPUIS LAB> |
| Template exploitable si membre ? | <A REMPLIR DEPUIS LAB> (oui si autres conditions ESC1/ESC3) |

## Pieges et OPSEC
- `Enrollment Rights` liste plusieurs principaux : ignorer `Domain Admins` / `Enterprise Admins` (par défaut) et se concentrer sur le groupe **spécifique** au template.
- Être dans le groupe ne suffit pas : vérifier aussi EKU + flags (`ENROLLEE_SUPPLIES_SUBJECT`, manager approval, signatures required).
- Trace : la simple énumération Certify est très discrète (LDAP read only), mais toute demande de cert derrière laisse trace Event **4886/4887**.

## Refs
- [[Cheatsheet - AD CS et certificats]]
- [[Synthèse - AD CS]]
- [[LO21 - AD CS (ESC1 et ESC3)]]
- [[Attaques AD]]

> [!note] Commande absente cheatsheet
> Le pipeline `Certify.exe find | Select-String ... -Context` et la requête `Get-DomainObjectAcl` sur templates AD CS ne sont pas dans [[Cheatsheet - AD CS et certificats]] — à ajouter en phase suivante.
