# Cheatsheet - LDAP SMB Kerberos

> [!summary] Objectif
> Regrouper les commandes utiles pour lire les objets AD, tester SMB et préparer les attaques Kerberos.

> [!warning] Cadre d'utilisation
> À utiliser uniquement dans un environnement de lab autorisé, dans le cadre de l'apprentissage et de la préparation CRTP.

## Quick use

- Enum users / computers
- Check shares
- Préparer SPN / preauth / delegation

## Core commands

### PowerView / AD module

```powershell
. C:\AD\Tools\PowerView.ps1
Get-DomainUser
Get-DomainUser -Identity student1
Get-DomainComputer | Select-Object Name
Get-ADUser -Filter * -Properties *
Get-ADComputer -Filter * | Select-Object Name
Get-ADDomain
Get-ADDomainController
```

Le bloc standard du PDF pour remonter users, computers et contexte de domaine.

### Search in user attributes

```powershell
Get-DomainUser -LDAPFilter "Description=*built*" | Select-Object name,Description
Get-ADUser -Filter 'Description -like "*built*"' -Properties Description | Select-Object name,Description
```

Très utile quand le lab cache des hints en `description`.

### SMB

```bash
smbclient -L //<DC_IP>/ -U <USER>
```

Valide rapidement l’auth SMB et les shares visibles.

### LDAP from Linux

```bash
ldapsearch -x -H ldap://<DC_IP> -b "DC=<DOMAIN>,DC=local"
```

Pratique si tu veux recouper la vue Linux avec la vue PowerShell.

## PDF notes

- Les `description`, `logonCount`, `pwdLastSet` et `ServicePrincipalName` reviennent souvent dans les labs.
- Si une sortie est trop large, repasse sur un attribut précis au lieu de garder `-Properties *`.

## Related

- [[CheetSheet]]
- [[Cheatsheet - Reconnaissance AD]]
- [[Cheatsheet - Kerberoasting]]
- [[Cheatsheet - Delegation attacks]]

