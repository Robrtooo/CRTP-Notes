# Cheatsheet - GPO et OU

> [!summary] Objectif
> Préparer les commandes utiles pour lister les OU, les GPO liées et les objets influencés dans les labs CRTP.

> [!warning] Cadre d'utilisation
> À utiliser uniquement dans un environnement de lab autorisé, dans le cadre de l'apprentissage et de la préparation CRTP.

## Quand utiliser cette fiche

- Quand un scénario mentionne une OU, une GPO ou des droits sur une policy.
- Quand il faut comprendre où une configuration de sécurité s'applique.

## Prérequis

- PowerView chargé dans une session PowerShell du lab.
- Le nom de l'OU ou de la GPO visée.

## Commandes

### Objectif

Lister les OU du domaine.

### Commande

```powershell
. C:\AD\Tools\PowerView.ps1
Get-DomainOU
Get-DomainOU | Select-Object -ExpandProperty name
```

### Explication

Affiche les unités d'organisation et permet de repérer rapidement la bonne branche d'objets.

### Contexte d’utilisation

Utile avant d'étudier les GPO appliquées à une OU précise.

### Précautions

Noter le `distinguishedName` de l'OU cible si plusieurs noms proches existent.

### Objectif

Lister les GPO et retrouver une policy précise.

### Commande

```powershell
Get-DomainGPO
Get-DomainGPO -Identity "DevOps Policy"
```

### Explication

Permet d'énumérer les GPO et d'interroger directement une policy par son nom.

### Contexte d’utilisation

Indispensable quand le lab demande d'analyser ou de modifier la portée d'une policy.

### Précautions

Bien distinguer le nom de la policy et son GUID.

### Objectif

Lister les ordinateurs d'une OU ciblée.

### Commande

```powershell
(Get-DomainOU -Identity DevOps).distinguishedname | %{Get-DomainComputer -SearchBase $_} | Select-Object name
```

### Explication

Commande directement issue du support pour recenser les machines placées dans l'OU `DevOps`.

### Contexte d’utilisation

Très utile pour relier une OU aux ordinateurs réellement concernés par une policy.

### Précautions

Conserver le nom exact de l'OU du lab si celui-ci diffère.

### Objectif

Relier une OU à sa GPO.

### Commande

```powershell
(Get-DomainOU -Identity DevOps).gplink
Get-DomainGPO -Identity (Get-DomainOU -Identity DevOps).gplink.substring(11,(Get-DomainOU -Identity DevOps).gplink.length-72)
```

### Explication

Récupère le `gplink` d'une OU puis résout la GPO correspondante, comme montré dans le PDF pour `DevOps Policy`.

### Contexte d’utilisation

Très utile pour relier droits sur OU, policy et impact sur les machines du lab.

### Précautions

Vérifier les OU et GPO sur lesquelles tu as réellement travaillé pour éviter les confusions dans les notes.

### Objectif

Recouper les ACL de GPO dans BloodHound.

### Commande

```text
BloodHound CE :
- Search for Applocker
- Inbound Object Control
- Search for DevOps
- Inbound Object Control
```

### Explication

Le lab manual s'appuie sur BloodHound CE pour lire plus vite les ACL des GPO `Applocked` et `DevOps`.

### Contexte d’utilisation

À utiliser après l'énumération PowerView pour valider le droit vu sur la policy.

### Précautions

Toujours noter si le droit observé est `GenericAll`, `WriteDACL` ou un autre contrôle précis.

### Notes associées

- [[Cheatsheet - ACL et droits AD]]
- [[Cheatsheet - Monitoring et détections]]
- [[Synthèse - ACL et GPO]]
- [[Active Directory - MOC]]
