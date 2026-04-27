### First is PowerView
### Second is ActiveDirectory Module
- Get current domain 
```
Get-Domain
Get-ADDomain 
```
- Get object of another domain
```
 Get-Domain -Domain moneycorp.local
 Get-ADDomain -Identity moneycorp.local
```
- Get domain SID for the current domain
```
Get-DomainSID
(Get-ADDomain).DomainSID
```
- Get domain controllers for the current domain
```
Get-DomainController
Get-ADDomainController
```
- Get domain controllers for another domain
```
Get-DomainController -Domain moneycorp.local
Get-ADDomainController -DomainName moneycorp.local -Discover
```
- Get information from Domain Policy
```
Get-DomainPolicyData
```
- Get a list of users in the current domain
```
Get-DomainUser
Get-DomainUser -Identity student1

Get-ADUser -Filter * -Properties *
Get-ADUser -Identity student1 -Properties *
```
- Get list of all properties for users in the current domain
```
Get-DomainUser -Identity student1 -Properties *
Get-DomainUser -Properties samaccountname, logonCount

Get-ADUser -Filter * -Properties * | select -First 1 | Get-Member -MemberType *Property | select Name
Get-ADUser -Filter * -Properties * | select name,logoncunt,@{expression={[datetime]::fromFileTime($_.pwdlastset)}} 
```


> [!DANGER] Logoncount
> If logoncount <= 10, le compte est dormant, ou que c'est un **decoy**.
> If logoncount > 10000, il y a un subway, service ou une schedul task associée **-> Vecteur D'ATTAQUE potentiellement important**

- Search for a particular string in a user's attributes 
```
Get-DomainUser -LDAPFilter "Description=*built*" | Select name,Description 

Get-ADUser -Filter 'Description -like "*built*"' -Properties Description | select name,Description
```


> [!NOTE] Information
> Sometimes, some attributes, password or service owned by the user are wrote in description.
> It can be really usefull to check it


- Get all the groups in the current domain-
```
Get-DomainGroup | select Name
Get-DomainGroup -Domain <targetdomain>

Get-ADGroup -Filter * | select Name
Get-ADGroup -Filter * -Properties
```
- Get all groups containing the word "admin" in group name
```
Get-DomainGroup *admin*

Get-ADGroup -Filter 'Name -like "*admin*"' | select Name
```
- 