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
 
```
- Get current domain 
```
 
```
- Get current domain 
```
 
```
- Get current domain 
```
 
```