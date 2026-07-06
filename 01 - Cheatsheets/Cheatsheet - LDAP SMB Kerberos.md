# Cheatsheet - LDAP SMB Kerberos

> [!summary] Objectif
> Lire les objets AD (users, computers, attributs), tester SMB et préparer les attaques Kerberos (SPN, preauth, delegation).

### Users / computers

```powershell
. C:\AD\Tools\PowerView.ps1
Get-DomainUser ; Get-DomainUser -Identity student1
Get-DomainComputer | Select-Object Name
Get-ADUser -Filter * -Properties *
```

### Search in user attributes

```powershell
Get-DomainUser -LDAPFilter "Description=*built*" | Select-Object name,Description
Get-ADUser -Filter 'Description -like "*built*"' -Properties Description | Select-Object name,Description
```

### SMB / LDAP from Linux

```bash
smbclient -L //<DC_IP>/ -U <USER>
ldapsearch -x -H ldap://<DC_IP> -b "DC=<DOMAIN>,DC=local"
```

> [!note] `description`, `logonCount`, `pwdLastSet`, `ServicePrincipalName` cachent souvent des hints. Sortie trop large → repasser sur un attribut précis au lieu de `-Properties *`.

## Related
[[CheetSheet]] · [[Cheatsheet - Reconnaissance AD]] · [[Cheatsheet - Kerberoasting]] · [[Cheatsheet - Delegation attacks]]
