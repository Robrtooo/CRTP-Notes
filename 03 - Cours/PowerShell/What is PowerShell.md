# What is PowerShell

> [!summary] Objectif
> Référence unique du savoir PowerShell du cours : ce que c'est, comment charger/exécuter du code, les mécanismes de détection Microsoft, et comment les analyser/contourner en lab. Les flashcards de révision : [[Flashcards - PowerShell]].

> [!note] Définition Microsoft
> "PowerShell is a cross-platform task automation solution made up of a command-line shell, a scripting language, and a configuration management framework."

Points clés :
- PowerShell (PS) est préinstallé sur tous les Windows modernes.
- PS **n'est pas** `powershell.exe` — c'est l'assembly `System.Management.Automation.dll`.
- On utilise Windows PowerShell (il existe aussi PS Core, multiplateforme).

## Chargement de scripts et modules

- Charger un script par **dot sourcing** :
![[Pasted image 20260211104154.png]]
*Chargement d'un script PowerShell par dot sourcing.*

- Importer un module pour exposer ses cmdlets (`Import-Module`) :
![[Pasted image 20260211104325.png]]
*Import d'un module PowerShell.*

## Exécution de code

- PS peut télécharger une chaîne et l'exécuter avec `iex` + `DownloadString` (download cradle) :
![[Pasted image 20260211105211.png]]
*Exécution d'un script chargé en mémoire (download cradle).*

- Plusieurs variantes existent (`iwr`, `Msxml2.XMLHTTP`, `System.IO.StreamReader`...) pour charger en mémoire sans écrire sur disque :
![[Pasted image 20260211105342.png]]
![[Pasted image 20260211105416.png]]
*Variantes de chargement en mémoire.*

Cette capacité de download-and-execute a poussé Microsoft à ajouter des détections (à partir de PowerShell v5).

## Mécanismes de détection Microsoft

Quatre mécanismes à connaître (comment Microsoft a réagi) :

- **System-Wide Transcription** — journalise toutes les entrées/sorties des sessions PS.
- **Script Block Logging** — journalise les blocs de code exécutés, même obfusqués/en mémoire.
- **AMSI** (AntiMalware Scan Interface) — soumet le contenu exécuté à l'antivirus au moment du run.
- **Constrained Language Mode (CLM)** — bride le langage (pas de `Add-Type`, réflexion limitée), intégré à AppLocker et WDAC (Device Guard).

## Execution Policy

![[Pasted image 20260211113826.png]]
*Rappel des politiques d'exécution PowerShell.*

- **Ce n'est pas** un contrôle de sécurité : ça empêche seulement l'exécution accidentelle de scripts.
- Facilement contournable pour exécuter des scripts.

![[Pasted image 20260211114825.png]]
*Exemple de variation de politique d'exécution.*

## Contourner les détections

### InviShell — éviter le logging
- Hooke l'**assembly .NET** plutôt que PowerShell lui-même.
- Utilise la **CLR Profiler API** pour poser le hook (le profiler DLL est chargé par le CLR au runtime).
- Résultat : exécution PS sans transcription ni script block logging.

### AMSITrigger / DefenderCheck — localiser ce qui est flaggé
Deux outils pour comprendre **quelle portion** d'un script/binaire déclenche la détection :
![[Pasted image 20260227124057.png]]

- **AMSITrigger** cible ce que déclenche AMSI ; **DefenderCheck** ce que flag Defender.
- Boucle type : scan avec AMSITrigger → modifier la portion flaggée → rescan → répéter jusqu'à `AMSI_RESULT_NOT_DETECTED`.
- Astuce lab : renommer le script avant de le scanner (sinon access denied).

### Obfuscation
> [!note] Invoke-Obfuscation
> Obfusque un script pour casser les signatures AMSI.
> ![[Pasted image 20260227124251.png]]

- **Invoke-Obfuscation** pour les scripts PowerShell ; **PowerUp** aide aussi à l'obfuscation (voir [[2. Good practices]]).
- Principe : moins de surface de code = analyse plus simple. Garder uniquement les fonctionnalités nécessaires.

## Bonnes pratiques de relecture (résumé)

Après passage des outils d'analyse, une relecture manuelle reste souvent nécessaire (voir [[2. Good practices]] pour le détail) :
- Revenir à la ligne/fonction signalée par l'outil.
- Identifier si le problème vient d'une chaîne, d'un bloc ou d'une fonctionnalité.
- Réécrire le minimum (ex. transformer une chaîne base64 flaggée et la reconstruire au runtime).

## Notes associées
- [[2. Good practices]]
- [[Using tools]]
- [[Flashcards - PowerShell]]
- [[Cheatsheet - PowerShell]]
- [[Tools list]]
- [[PowerShell - MOC]]
