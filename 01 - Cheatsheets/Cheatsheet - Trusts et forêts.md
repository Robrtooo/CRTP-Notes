# Cheatsheet - Trusts et forêts

> [!summary] Objectif
> Préparer l’énumération des domaines, forêts et relations de confiance avant les modules cross-trust.

> [!warning] Cadre d'utilisation
> À utiliser uniquement dans un environnement de lab autorisé, dans le cadre de l'apprentissage et de la préparation CRTP.

## Quick use

- `WITHIN_FOREST`
- `FILTER_SIDS`
- `eurocorp.local`
- external trust vs intra-forest trust

## Core commands

### PowerView view

```powershell
. C:\AD\Tools\PowerView.ps1
Get-DomainTrust
Get-ForestDomain | %{Get-DomainTrust -Domain $_.Name}
Get-DomainTrust | Where-Object {$_.TrustAttributes -eq "FILTER_SIDS"}
Get-ForestDomain | %{Get-DomainTrust -Domain $_.Name} | Where-Object {$_.TrustAttributes -eq "FILTER_SIDS"}
Get-ForestDomain -Forest eurocorp.local | %{Get-DomainTrust -Domain $_.Name}
```

Le bloc PowerView du PDF pour lire les trusts et distinguer les trusts externes.

### AD module view

```powershell
Import-Module C:\AD\Tools\ADModulemaster\Microsoft.ActiveDirectory.Management.dll
Import-Module C:\AD\Tools\ADModulemaster\ActiveDirectory\ActiveDirectory.psd1
(Get-ADForest).Domains
Get-ADTrust -Filter *
Get-ADTrust -Filter '(intraForest -ne $True) -and (ForestTransitive -ne $True)'
Get-ADTrust -Filter * -Server eurocorp.local
```

Version AD module montrée juste après dans le support.

## PDF notes

- `FILTER_SIDS` = pivot visuel important dans les exemples cross-forest.
- Si PowerView lève une referral error en trust externe non transitive, le PDF l’explique explicitement.

## Related

- [[Cheatsheet - Delegation attacks]]
- [[Cheatsheet - AD CS et certificats]]
- [[Synthèse - Trusts et forêts]]

