---
flag: 6
machine: Student VM
statut: validé
technique: Hunt derivative admin (PSRemoting)
LO: [[LO06 - Derivative local admin]]
---

# F06 - Script hunt admin via PS Remoting

> [!summary] Objectif
> Identifier le script du toolkit `C:\AD\Tools` qui liste les hôtes où l identité courante est admin local **via PSRemoting**.

## Contexte
- **Etape attack flow**: après privesc / avec un compte AD → chasser la **dérivée d admin local** (les box où tu peux ouvrir une session sans autre credential).
- **Ce qu on a**: un shell PowerShell en tant que `studentX` (ou une identité relayée via OverPass-the-Hash).
- **Ce qu on cherche**: le **nom exact du script** (réponse du flag) qui teste l accès admin PSRemoting sur chaque machine du domaine.

## Solution

### Etape 1 - Repérer le toolkit
```powershell
Get-ChildItem C:\AD\Tools\ -Filter *PSRemoting* -Recurse
```
Filtre les scripts contenant `PSRemoting`. Le script utile est un one-shot signé Kevin Robertson qui enchaîne `Get-DomainComputer` + test `Enter-PSSession`.

### Etape 2 - Exécuter le hunt
```powershell
. C:\AD\Tools\Find-PSRemotingLocalAdminAccess.ps1
Find-PSRemotingLocalAdminAccess
```
Le script parcourt tous les hôtes du domaine et retourne ceux où `whoami` renvoie un token admin via WinRM (5985). Les cibles listées sont accessibles directement en `Enter-PSSession -ComputerName <TARGET>`.
→ Voir [[Cheatsheet - Lateral movement]] §"Après Find-PSRemotingLocalAdminAccess".

## Reponse

| Element | Valeur |
|---------|--------|
| Script hunt PSRemoting | Find-PSRemotingLocalAdminAccess.ps1 |

## Pieges et OPSEC
- Chaque test = **une connexion WinRM sortante** vers chaque machine du domaine → très bruyant (events 4624 logon type 3 côté cible, 4648 côté source).
- **Double-hop**: une fois dans `Enter-PSSession`, tes creds ne franchissent pas un 2e saut. Ne relance pas le hunt depuis la session distante → utiliser OverPass-the-Hash ou `-Credential` en local.
- L identité courante compte : si tu chasses en tant que `studentX`, tu ne verras que **ses** dérivées. Rejoue après OverPass-the-Hash pour élargir.

## Refs
- [[Cheatsheet - Lateral movement]]
- [[Cheatsheet - Privilege escalation]]
- [[Cheatsheet - Reconnaissance AD]]
- [[LO06 - Derivative local admin]]
- [[Flags - Index]] · [[Journal - Index]]
