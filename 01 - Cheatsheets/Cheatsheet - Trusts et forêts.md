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

> [!note] `FILTER_SIDS` conditionne ce qu'un trust forgé permet d'obtenir : sans ce flag, injecter le SID d'Enterprise Admins dans `SIDHistory` d'un ticket forgé donne directement les droits EA de l'autre côté du trust. Avec le flag, cette astuce est bloquée côté cible et il faut passer par un referral ticket propre pour traverser.

### AD module view

```powershell
Import-Module C:\AD\Tools\ADModulemaster\ActiveDirectory\ActiveDirectory.psd1
(Get-ADForest).Domains
Get-ADTrust -Filter *
Get-ADTrust -Filter '(intraForest -ne $True) -and (ForestTransitive -ne $True)'
```

Une referral error sur un trust externe non transitif est attendue (le PDF l'explique) — abus (trust key, referral ticket) détaillés dans [[1. Cheatsheet - Toutes les commandes]].

> [!warning]- Required (abus de trust)
> Privilèges Domain Admin d'un côté du trust, pour en extraire la clé (`lsadump::evasive-trust`). Voir [[Cheatsheet - Tickets Kerberos]] pour obtenir DA au préalable.

**Link** — [[Cheatsheet - Delegation attacks]] · [[Cheatsheet - AD CS et certificats]] · [[Synthèse - Trusts et forêts]]
