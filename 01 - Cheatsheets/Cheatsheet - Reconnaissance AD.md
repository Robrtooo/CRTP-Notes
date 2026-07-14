# Cheatsheet - Reconnaissance AD

> [!summary] Objectif
> First-pass recon : identifier domaine, SID, DC, policy et services AD exposés, avant LDAP / SMB / Kerberos / BloodHound. Début de lab, pour valider la cible.

> [!abstract] Pourquoi · Chaînage
> Couche **identité (AD)** — carte de départ, **pas d'exploit**. **Repérer** domaine, SID, DC, policy.
> **Prérequis** un compte de domaine → **débloque** tout le reste (users, ACL, kerberoast, BloodHound).

### DNS / basic network

```bash
nslookup <DOMAIN>
nslookup <DC_IP>
nmap -sV -Pn -p 53,88,135,139,389,445,464,636,3268,3269 <DC_IP>
```

### PowerView - current domain

```powershell
. C:\AD\Tools\PowerView.ps1
Get-Domain
Get-DomainSID
Get-DomainController
Get-DomainPolicyData
```

### AD module - same checks

```powershell
Get-ADDomain
(Get-ADDomain).DomainSID
Get-ADDomainController
Get-ADGroupMember -Identity "Enterprise Admins" -Server moneycorp.local   # membres du groupe via AD module (flag F01)
```

### Session / local admin hunting sur une cible précise (avant delegation abuse)

```powershell
# Voir qui est admin local sur un serveur unconstrained (flag F24)
Find-LocalAdminAccess -ComputerName dcorp-appsrv
Invoke-UserHunter -ComputerName dcorp-appsrv
Get-NetLocalGroupMember -ComputerName dcorp-appsrv -GroupName Administrators
```

### WMI service query (identifier un process/service qui tourne sous un compte cible)

```powershell
# Trouver l'hôte où un compte (ex svcadmin DA) est utilisé par un service (flag F10)
Get-WmiObject -Class Win32_Service -ComputerName <HOTE_CIBLE> `
  | Where-Object { $_.StartName -like "*<user>*" } `
  | Select-Object Name,StartName,PathName
```

### Another domain in lab

```powershell
Get-Domain -Domain moneycorp.local
Get-ADDomainController -DomainName moneycorp.local -Discover
```

> [!note] PowerView et AD Module sont les deux pivots montrés en parallèle. Garder domaines et DC séparés dès qu'on passe en multi-domaines / forêts.

## Lecture LDAP / SMB / Kerberos

> [!abstract] Pourquoi · Chaînage
> Couche **identité (AD)** — lecture brute des objets. **Repérer** users/computers/attributs (SPN, preauth, `description` = hints).
> **Prérequis** compte de domaine → **débloque** Kerberoast (SPN), AS-REP (preauth), delegation.

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

## Voir aussi
- [[Cheatsheet - BloodHound SharpHound]]
- [[Cheatsheet - Kerberoasting]]
- [[Cheatsheet - AS-REP Roasting]]
- [[Cheatsheet - Delegation attacks]]
- [[Cheatsheet - PowerShell]]
