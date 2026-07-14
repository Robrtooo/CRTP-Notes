# Cheatsheet - Chemins des outils (`C:\AD\Tools`)

> [!info] Map outil → chemin (VM student CRTP)
> Racine `C:\AD\Tools\` = versions **canoniques**. `Sliver\` = arsenal élargi (suite `Sharp*`, coercion, deleg). `Old_Tools\` / `BloodHound-4.0.3_old\` = obsolètes, éviter.
> Rappel lancement : `.ps1` → `. .\x.ps1` (dot-source) · `.exe` → `.\x.exe`.

## Collecte / Recon
| Outil | Chemin |
|---|---|
| PowerView | `C:\AD\Tools\PowerView.ps1` |
| SharpView | `C:\AD\Tools\Sliver\SharpView.exe` |
| **SharpHound (collector .exe)** | `C:\AD\Tools\BloodHound-master\BloodHound-master\Collectors\SharpHound.exe` |
| SharpHound (collector .ps1) | `C:\AD\Tools\BloodHound-master\BloodHound-master\Collectors\SharpHound.ps1` |
| SharpHound (raccourci Sliver) | `C:\AD\Tools\Sliver\SharpHound.exe` |
| SOAPHound (ADWS, furtif) | `C:\AD\Tools\SOAPHound.exe` |
| SharpADWS | `C:\AD\Tools\Sliver\SharpADWS.exe` |
| ADCollector / ADSearch | `C:\AD\Tools\Sliver\ADCollector.exe` · `...\Sliver\ADSearch.exe` |
| Seatbelt | `C:\AD\Tools\Sliver\Seatbelt.exe` |
| AD module (RSAT) | `C:\AD\Tools\ADModule-master\` → `Import-Module .\Microsoft.ActiveDirectory.Management.dll` |

> [!warning] BloodHound GUI ≠ collector
> `BloodHound-win32-x64\` = juste l'appli de visualisation. Le collector = les chemins ci-dessus (double `BloodHound-master`).

## ACLs
| Outil | Chemin |
|---|---|
| FindInterestingDomainACL | `C:\AD\Tools\Sliver\FindInterestingDomainACL.exe` |
| FindACLs / ACLFinder | `C:\AD\Tools\Sliver\FindACLs.exe` · `...\ACLFinder.exe` |
| RACE (ACL persistence) | `C:\AD\Tools\RACE.ps1` |
| Invoke-SDPropagator | `C:\AD\Tools\Invoke-SDPropagator.ps1` |
| Set-ADACL | `C:\AD\Tools\Old_Tools\Set-ADACL.ps1` |

## Kerberos (roast / tickets / deleg)
| Outil | Chemin |
|---|---|
| **Rubeus** | `C:\AD\Tools\Rubeus.exe` |
| GetUserSPNs (kerberoast) | `C:\AD\Tools\kerberoast\GetUserSPNs.ps1` |
| Get-RBCD-Threaded | `C:\AD\Tools\Get-RBCD-Threaded.exe` |
| StandIn (RBCD/attrs) | `C:\AD\Tools\Sliver\StandIn.exe` |
| Whisker (shadow creds) | `C:\AD\Tools\Sliver\Whisker.exe` |

## Creds / LSASS / secrets
| Outil | Chemin |
|---|---|
| **mimikatz** | `C:\AD\Tools\mimikatz.exe` |
| SafetyKatz | `C:\AD\Tools\SafetyKatz.exe` |
| SharpKatz | `C:\AD\Tools\Sliver\SharpKatz.exe` |
| Invoke-Mimi | `C:\AD\Tools\Invoke-Mimi.ps1` |
| Invoke-TheKat | `C:\AD\Tools\Invoke-TheKat.ps1` |
| FindLSASSPID | `C:\AD\Tools\FindLSASSPID.exe` |
| minidumpdotnet | `C:\AD\Tools\minidumpdotnet.exe` |
| SharpDPAPI | `C:\AD\Tools\Sliver\SharpDPAPI.exe` |
| SharpSecDump | `C:\AD\Tools\Sliver\SharpSecDump.exe` |
| GMSAPasswordReader | `C:\AD\Tools\Sliver\GMSAPasswordReader.exe` |
| LAPSDecrypt | `C:\AD\Tools\Sliver\LAPSDecrypt.exe` |

## Privesc locale
| Outil | Chemin |
|---|---|
| PowerUp | `C:\AD\Tools\PowerUp.ps1` |
| SharpUp | `C:\AD\Tools\Sliver\SharpUp.exe` |
| PrivEscCheck | `C:\AD\Tools\PrivEscCheck.ps1` |
| winPEAS | `C:\AD\Tools\winPEASx64.exe` |
| WeakServiceEnum / accesschk | `C:\AD\Tools\Sliver\WeakServiceEnum.exe` · `...\accesschk64.exe` |

## Lateral movement / admin hunting
| Outil | Chemin |
|---|---|
| **Find-PSRemotingLocalAdminAccess** | `C:\AD\Tools\Find-PSRemotingLocalAdminAccess.ps1` |
| Find-WMILocalAdminAccess | `C:\AD\Tools\Find-WMILocalAdminAccess.ps1` |
| Invoke-SessionHunter | `C:\AD\Tools\Invoke-SessionHunter.ps1` |
| LACheck | `C:\AD\Tools\Sliver\LACheck.exe` |
| SharpWMI / CSharpWinRM | `C:\AD\Tools\Sliver\SharpWMI.exe` · `...\CSharpWinRM.exe` |
| WSManWinRM | `C:\AD\Tools\WSManWinRM.exe` |
| nxc (netexec) | `C:\AD\Tools\Sliver\nxc.exe` |

## Coercion (printer bug / relay)
| Outil | Chemin |
|---|---|
| MS-RPRN | `C:\AD\Tools\MS-RPRN.exe` |
| SpoolSample | `C:\AD\Tools\Sliver\SpoolSample.exe` |
| DFSCoerce | `C:\AD\Tools\DFSCoerce-andrea.exe` |
| WSPCoerce | `C:\AD\Tools\WSPCoerce.exe` |

## GPO
| Outil | Chemin |
|---|---|
| SharpGPOAbuse | `C:\AD\Tools\SharpGPOAbuse.exe` |
| GPOMemberAudit | `C:\AD\Tools\Sliver\GPOMemberAudit.exe` |
| GPOddity | `C:\AD\Tools\GPOddity\` |

## ADCS
| Outil | Chemin |
|---|---|
| Certify | `C:\AD\Tools\Certify.exe` |

## MSSQL
| Outil | Chemin |
|---|---|
| PowerUpSQL | `C:\AD\Tools\PowerUpSQL-master\PowerUpSQL.ps1` |
| HeidiSQL | `C:\AD\Tools\HeidiSQL_9.4_Portable\` |

## Visualisation BloodHound
| Élément | Chemin |
|---|---|
| neo4j (DB) — **double-niché** | `C:\AD\Tools\neo4j-community-4.4.5-windows\neo4j-community-4.4.5\bin\neo4j.bat console` |
| BloodHound GUI | `C:\AD\Tools\BloodHound-win32-x64\` (vérifier `BloodHound.exe` niché) |
| neo4j browser | http://localhost:7474 (login `neo4j/neo4j` au 1er lancement) |

## Évasion / OPSEC
| Outil | Chemin |
|---|---|
| InviShell (non-admin) | `C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat` |
| AmsiTrigger | `C:\AD\Tools\AmsiTrigger_x64.exe` |
| DefenderCheck | `C:\AD\Tools\DefenderCheck.exe` |
| Invoke-EDRChecker / SharpEDRChecker | `C:\AD\Tools\Invoke-EDRChecker.ps1` · `...\Sliver\SharpEDRChecker.exe` |
| Stracciatella (PS sans AMSI/logging) | `C:\AD\Tools\Sliver\Stracciatella.exe` |
| Amsi bypass (snippets) | `C:\AD\Tools\Amsi-Byp.txt` |

## Shells / transfert / C2
| Outil | Chemin |
|---|---|
| Invoke-PowerShellTcp | `C:\AD\Tools\Invoke-PowerShellTcp.ps1` |
| netcat | `C:\AD\Tools\netcat-win32-1.12\nc64.exe` |
| HFS (file server) | `C:\AD\Tools\hfs.exe` |
| Loader / NetLoader | `C:\AD\Tools\Loader.exe` · `...\Old_Tools\NetLoader.exe` |
| Sliver C2 | `C:\AD\Tools\Sliver\` |

Voir aussi [[1. Cheatsheet - Toutes les commandes]] · [[Cheatsheet - BloodHound SharpHound]]
