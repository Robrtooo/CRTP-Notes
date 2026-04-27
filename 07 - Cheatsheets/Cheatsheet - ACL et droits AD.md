# Cheatsheet - ACL et droits AD

> [!summary] Objectif
> Préparer une fiche sur l'énumération des ACL Active Directory, des droits intéressants et des vérifications BloodHound associées.

> [!warning] Cadre d'utilisation
> À utiliser uniquement dans un environnement de lab autorisé, dans le cadre de l'apprentissage et de la préparation CRTP.

## Quand utiliser cette fiche

- Quand une étape du lab parle de `WriteDACL`, `GenericAll`, `ForceChangePassword` ou de contrôle d'objet.
- Quand il faut relier PowerView et BloodHound sur un même objet AD.

## Prérequis

- PowerView chargé dans une session PowerShell du lab.
- Un utilisateur ou groupe cible à inspecter.

## Commandes

### Objectif

Lister les ACL d'un objet sensible.

### Commande

```powershell
. C:\AD\Tools\PowerView.ps1
Get-DomainObjectAcl -Identity "Domain Admins" -ResolveGUIDs -Verbose
```

### Explication

Permet d'énumérer les ACE d'un objet et de résoudre les GUID vers des droits lisibles.

### Contexte d’utilisation

Très utile pour comprendre quelles permissions sont réellement accordées sur un groupe ou une GPO.

### Précautions

Documenter l'objet, l'identité observée et le droit exact avant d'aller plus loin.

### Objectif

Rechercher des ACL intéressantes pour un utilisateur compromis.

### Commande

```powershell
Find-InterestingDomainAcl -ResolveGUIDs | Where-Object {$_.IdentityReferenceName -match "studentx"}
Find-InterestingDomainAcl -ResolveGUIDs | Where-Object {$_.IdentityReferenceName -match "RDPUsers"}
```

### Explication

Reprend la séquence du lab manual pour vérifier d'abord l'utilisateur compromis, puis un groupe auquel il appartient.

### Contexte d’utilisation

Utile quand le lab demande de partir d'un compte standard pour trouver un droit mal configuré.

### Précautions

Ne pas confondre présence d'un droit et exploitabilité réelle sans vérifier la cible précise.

### Objectif

Recouper les ACL avec BloodHound.

### Commande

```text
BloodHound CE / Legacy :
- Rechercher l'utilisateur
- Ouvrir "Outbound Object Control"
- Examiner les droits sur groupes, GPO et ordinateurs
```

### Explication

BloodHound aide à visualiser les contrôles d'objet plus vite que la sortie brute des cmdlets.

### Contexte d’utilisation

Idéal pour vérifier visuellement les contrôles d'objet montrés par le PDF, par exemple sur `Applocked` ou `DevOps Policy`.

### Précautions

Conserver la correspondance entre l'objet vu dans BloodHound et la sortie PowerShell.

### Notes associées

- [[Enumeration]]
- [[Cheatsheet - BloodHound SharpHound]]
- [[Cheatsheet - GPO et OU]]
- [[Synthèse - ACL et GPO]]
