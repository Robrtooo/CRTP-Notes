# What is PowerShell

> [!summary] Résumé
> Introduction à PowerShell dans le contexte CRTP : définition, chargement de scripts, exécution, détections et points d'attention en lab.

> [!warning] Cadre d'utilisation
> À utiliser uniquement dans un environnement de lab autorisé, dans le cadre de l'apprentissage et de la préparation CRTP.

> [!NOTE] Microsoft Def
> "PowerShell is a cross-platform task automation solution made up of a command-line shell, a scripting language, and a configuration management framework."

PowerShell will now be call PS

- PS comes installed by-default on all the modern Windows OS.
- PS is NOT `powershell.exe`.
- It is the `System.Management.Automation.dll`.
- We will use Windows PowerShell.
- NB: There is a platform independant PS Core as well.

## PS Scripts and Modules

- Load a PS script using dot sourcing
![[Pasted image 20260211104154.png]]

*Légende : chargement d'un script PowerShell par dot sourcing.*

> [!note] Importing module or script
> This is important to understand how PowerShell loads functions and modules in a lab workflow.
> Using `Import-Module` :
> ![[Pasted image 20260211104325.png]]

*Légende : import d'un module PowerShell pour exposer ses cmdlets.*

## PS Script Execution

- It is possible de make PS download a string and execute it using `iex` and `DownloadString`.

![[Pasted image 20260211105211.png]]

*Légende : exemple pédagogique d'exécution d'un script chargé en mémoire.*

It forced Microsoft to react by implementing additional detections.
Known as MZ.
-> Introduced in Windows PS 5.

- To exec PS Script into memory
![[Pasted image 20260211105342.png]]
![[Pasted image 20260211105416.png]]

*Légende : variantes de chargement en mémoire dans un contexte de démonstration de lab.*

## PS Detections

* How did Microsoft react?
  * System-Wide Transcription
  * Script Block logging
  * AntiMalware Scan Interface (AMSI)
  * Constrained Language Mode (CLM) integrated with Applocker and WDAC

## Execution Policy

![[Pasted image 20260211113826.png]]

*Légende : rappel des politiques d'exécution PowerShell.*

- Not a security control
- Prevent user to execute unintended scripts
- Possible to bypass to exec scripts in a lab scenario

![[Pasted image 20260211114825.png]]

*Légende : exemple de variation de politique d'exécution.*

## Étude des contrôles de sécurité PowerShell

* Use **Invisi Shell**
* Hook .NET Assembly instead of PS
* Use CLR Profiler API to perform the hook

-> In the course material, these points are studied to understand how protections and detections behave in the lab.

## Outils d'analyse mentionnés

There are multiple tools that help evaluate what Defender may flag as malicious in a controlled lab.

* AMSITrigger
* DefenderCheck

These tools are used to inspect scripts or binaries and understand what is being flagged.

> [!NOTE] How to use
> ![[Pasted image 20260227124057.png]]

> [!NOTE] How to obfuscate
> Use tool Invoke-Obfuscation
> ![[Pasted image 20260227124251.png]]

## Points d'attention

It can be useful to keep only the functionalities needed in the code.
-> Less code surface often means simpler analysis during the lab.

## Notes associées

- [[PowerShell - MOC]]
- [[Cheatsheet - PowerShell]]
- [[2. Good practices]]
