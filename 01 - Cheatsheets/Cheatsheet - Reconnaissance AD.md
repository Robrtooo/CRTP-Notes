# Cheatsheet - Reconnaissance AD

> [!summary] Objectif
> First-pass recon pour identifier domaine, DC et services utiles avant de partir sur LDAP / SMB / Kerberos.

> [!warning] Cadre d'utilisation
> À utiliser uniquement dans un environnement de lab autorisé, dans le cadre de l'apprentissage et de la préparation CRTP.

## Quick use

- Début de lab
- Validation du domaine cible
- Base de départ avant PowerView / BloodHound

## Core commands

### DNS / basic network

```bash
nslookup <DOMAIN>
nslookup <DC_IP>
nmap -sV -Pn -p 53,88,135,139,389,445,464,636,3268,3269 <DC_IP>
```

Valide la résolution DNS et les services AD exposés sur le contrôleur de domaine.

### PowerView - domaine courant

```powershell
. C:\AD\Tools\PowerView.ps1
Get-Domain
Get-DomainSID
Get-DomainController
Get-DomainPolicyData
```

Le bloc de base du PDF pour confirmer domaine, SID, DC et policy.

### AD module - same checks

```powershell
Get-ADDomain
(Get-ADDomain).DomainSID
Get-ADDomainController
```

Version signed / Microsoft-friendly des mêmes vérifications.

### Another domain in lab

```powershell
Get-Domain -Domain moneycorp.local
Get-ADDomain -Identity moneycorp.local
Get-DomainController -Domain moneycorp.local
Get-ADDomainController -DomainName moneycorp.local -Discover
```

Utile quand le PDF bascule vers plusieurs domaines / forêts.

## PDF notes

- `PowerView` et `AD Module` sont les deux pivots d’énumération montrés en parallèle.
- Garde les noms de domaine / DC séparés dans tes notes de lab.

## Related

- [[Enumeration]]
- [[CheetSheet]]
- [[Cheatsheet - LDAP SMB Kerberos]]
- [[Cheatsheet - BloodHound SharpHound]]

