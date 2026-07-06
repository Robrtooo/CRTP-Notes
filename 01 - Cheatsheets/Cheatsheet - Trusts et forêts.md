# Cheatsheet - Trusts et forêts

> [!summary] Objectif
> Énumérer domaines, forêts et relations de confiance avant les modules cross-trust. Distinguer intra-forest / external, repérer `FILTER_SIDS`.

### PowerView view
```powershell
. C:\AD\Tools\PowerView.ps1
Get-DomainTrust
Get-ForestDomain | %{Get-DomainTrust -Domain $_.Name}
Get-DomainTrust | Where-Object {$_.TrustAttributes -eq "FILTER_SIDS"}
Get-ForestDomain -Forest eurocorp.local | %{Get-DomainTrust -Domain $_.Name}
```

### AD module view
```powershell
Import-Module C:\AD\Tools\ADModulemaster\ActiveDirectory\ActiveDirectory.psd1
(Get-ADForest).Domains
Get-ADTrust -Filter *
Get-ADTrust -Filter '(intraForest -ne $True) -and (ForestTransitive -ne $True)'
```

> [!note] `FILTER_SIDS` = pivot clé en cross-forest. Une referral error sur un trust externe non transitif est attendue (le PDF l'explique). Abus (trust key, referral ticket) → [[1. Cheatsheet - Toutes les commandes]].

## Related
[[Cheatsheet - Delegation attacks]] · [[Cheatsheet - AD CS et certificats]] · [[Synthèse - Trusts et forêts]]
