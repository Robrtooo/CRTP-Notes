# Tools list

> [!summary] Objectif
> Tous les outils cités dans le cours et le lab CRTP (LabManual + slides), regroupés par fonction, avec en une ligne ce que chacun permet de faire. Les invocations exactes sont dans [[1. Cheatsheet - Toutes les commandes]].

## Énumération

- **PowerView.ps1** — énumération AD via LDAP (users, groups, ACL, GPO, trusts, sessions, delegation) ; la brique centrale de la reconnaissance.
- **ActiveDirectory Module** (`Microsoft.ActiveDirectory.Management.dll`, `ActiveDirectory.psd1`) — cmdlets AD signées Microsoft, fonctionnent même en Constrained Language Mode.
- **SharpHound.exe / BloodHound** — collecteur + interface graphe : calcule les chemins d'attaque (shortest path to DA, object control).
- **SOAPHound.exe** — énumération furtive via ADWS (port 9389) au lieu de requêtes LDAP directes, comme l'AD module.
- **Invoke-SessionHunter.ps1** — repère les sessions utilisateur (dont DA) ouvertes sur les machines distantes.
- **Find-PSRemotingLocalAdminAccess.ps1** — liste les hôtes où le compte courant est admin local via PSRemoting.
- **Find-WMILocalAdminAccess.ps1** — même chose via WMI.

## Privilege escalation locale

- **PowerUp.ps1** — audit des mauvaises configs Windows exploitables (`Invoke-AllChecks`) + fonctions d'abus (`Invoke-ServiceAbuse`).
- **winPEASx64.exe** — audit de privesc locale exhaustif (services, chemins, creds, tâches).
- **PrivEscCheck.ps1** — audit de privesc locale orienté checklist.

## Loader & évasion

- **Loader.exe** (NetLoader) — charge et exécute un binaire .NET en mémoire depuis un fichier ou une URL, patche AMSI & ETW ; version obfusquée dans le lab.
- **InviShell** (`RunWithRegistryNonAdmin.bat`, `InShellProf.dll`) — hooke le .NET via la CLR Profiler API pour exécuter PowerShell sans transcription ni script block logging.
- **AMSITrigger / AmsiTrigger_x64.exe** — identifie la portion précise d'un script qui déclenche AMSI.
- **DefenderCheck.exe** — identifie la portion d'un binaire/script que Defender flag.
- **Invoke-Obfuscation** — obfuscation de scripts PowerShell pour casser les signatures AMSI.
- **Codecepticon.exe** — obfuscation du code source C#/.NET avant compilation.
- **ConfuserEx** — obfuscation d'un binaire .NET déjà compilé.
- **CSWhispers** — remplace les WinAPI par leurs équivalents NT (syscalls directs) pour l'évasion.
- **WSManWinRM.exe** — exécution WinRM en contournant certaines règles ASR.

## Credential access & tickets Kerberos

- **Rubeus.exe** — couteau suisse Kerberos : `asktgt`, `kerberoast`, `asreproast`, `s4u`, `golden`, `silver`, `diamond`, `monitor`, `dcsync`.
- **SafetyKatz.exe** — Mimikatz porté en .NET (build obfusqué `evasive-*`) : `sekurlsa`, `lsadump::lsa/sam/dcsync/trust`.
- **Mimikatz / Invoke-Mimikatz / Invoke-Mimi.ps1** — extraction de credentials, ekeys, tickets, DCSync (référence historique, très détectée).
- **minidumpdotnet.exe** — dump LSASS via .NET (évite les .dmp corrompus par certains EDR).
- **FindLSASSPID.exe** — récupère le PID de LSASS pour le dump.
- **Get-PassHashes.ps1** — extraction de hashes locale.

## Coercition d'authentification

- **MS-RPRN.exe** — Printer Bug : force un DC à s'authentifier vers un hôte contrôlé (via le service Print Spooler).
- **WSPCoerce.exe** — coercition via MS-WSP (Windows Search Protocol).
- **DFSCoerce-andrea.exe** — coercition via MS-DFSNM (service DFS).

## AD CS

- **Certify.exe** — énumère les CA et templates de certificats, détecte et abuse les templates vulnérables (ESC1, ESC3).

## MSSQL

- **PowerUpSQL** (`PowerupSQL.psd1`) — découverte des instances SQL du domaine et crawl/abus des database links (`Get-SQLServerLinkCrawl`).

## Persistance & ACL

- **RACE.ps1** — pose des security descriptors pour l'accès distant (WMI, PSRemoting) sans droits admin.
- **Invoke-SDPropagator.ps1** — force l'exécution de SDProp (persistance via AdminSDHolder).

## Reverse shell & infrastructure

- **Invoke-PowerShellTcp.ps1** (Nishang) — reverse shell PowerShell.
- **nc64.exe** — netcat Windows (listener de reverse shell).
- **hfs.exe** (HTTP File Server) — héberge les payloads sur la VM student pour les download-cradles.
- **openssl.exe** — conversion des certificats (PEM → PFX) dans le workflow AD CS.
- **john.exe** (John the Ripper) — cracking offline des hashes (Kerberoast, AS-REP).

## Voir aussi
- [[1. Cheatsheet - Toutes les commandes]] · [[Cheatsheet - PowerShell]] · [[What is PowerShell]]
