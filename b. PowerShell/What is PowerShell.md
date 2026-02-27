[]()
> [!NOTE] Microsoft Def
> "PowerShell is a cross-platform task automation solution made up of a command-line shell, a scripting language, and a configuration management framework."
> 

****
**PowerShell will now be call PS**

- PS comes installed by-default on all the modern Windows OS.
--> Such as VBS.

- **PS is NOT powershell.exe.** 
**--> It is the System.Management.Automation.dll**

- We will use Windows PowerShell. 
NB: There is a platform independant PS Core as well.

**** 
### PS Scripts and Modules

- Load a PS script using dot sourcing
	- ![[Pasted image 20260211104154.png]]

> [!danger] Importing module or script
> This is **SUPER IMPORTANT** to understand and abuse PS.
> Using `Import-Module` : 
> ![[Pasted image 20260211104325.png]]

*****
### PS Script Execution

- It is possible de make PS download a string, that we can abuse  to make it execute a potential payload, using `iex` and `DownloadString`

![[Pasted image 20260211105211.png]]

It **forced** Microsoft to react implementing a Malware Detector in fees.
Known as MZ.
-> Introduced in Windows PS 5.

- To exec PS Script into memory
![[Pasted image 20260211105342.png]]
![[Pasted image 20260211105416.png]]

### PS Detections
* **How Did Microsoft reacted  ?**
	* System-Wide Transcription
	-> All PS commands are now logged (chiffré). Enabled by default
	* Script Block logging
	-> Enabled by default. Log any suspicious command and related infs.
	* AntiMalware Scan Interface (AMSI)
	-> B4 exec of script, will pick the result content of command and send it to an analyser to determine legitimity.
	* Constrained Language Mode (CLM) - Integrated with Applocker
	and WDAC (Device Guard)


### Execution Policy
![[Pasted image 20260211113826.png]]

- ! Not a security control
- Prevent user to execute unintended scripts
- Possible to bypass to exec scripts.

![[Pasted image 20260211114825.png]]


### Bypass Security security controls PS

* Use **Invisi Shell**
* Hook .NET Assembly instead of PS
* Use CLR Profiler API to perform the hook


-> To avoid detection, you can still load your script from memory to perform AMSI bypass


### BE SURE ABOUT WHAT YOU DO !!


Their are multiple tool that call help you to see what Defender may flag as malicious.

* AMSITrigger
* DefenderCheck

These tool will scan binaries to see if it'll be flag.
You can still try to edit these and perform analysis again till it not get flag as malicious.


> [!NOTE] How to use
> ![[Pasted image 20260227124057.png]]


> [!NOTE] How to obfuscate 
> Use tool Invoke-Obfuscation
> ![[Pasted image 20260227124251.png]]


## NB !!!


> [!Danger] IMPORTANT
[> It can be super usefull to keep only the fonctionnalities we need in the code.]()
> -> Less chance to get flagged / Signature will change

	[[[2. Good practices ]] 
	
