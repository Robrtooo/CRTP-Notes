---
flag: 8
machine: dcorp-ci
statut: validé
technique: Jenkins RCE (build step / Groovy)
LO: "[[LO05 - Privesc locale et Jenkins]]"
---

# F08 - User de domaine du service Jenkins

> [!summary] Objectif
> Identifier le compte de domaine sous lequel s exécute le service Jenkins sur `dcorp-ci` (format `dcorp\xxx`).

## Contexte
- **Etape attack flow**: authentifié sur Jenkins (F07) → exécuter du code → l identité renvoyée par `whoami` = compte de service = point de pivot LO7.
- **Ce qu on a**: session Jenkins active (voie A ou B de F07) sur `http://<dcorp-ci>:8080`.
- **Ce qu on cherche**: le **user de domaine** qui exécute le service Jenkins (réponse du flag). Ce compte est admin **High** sur `dcorp-ci` et souvent admin dérivé sur `dcorp-mgmt`.

## Solution

### Etape 1 - Voie build step (droit Configure sur un job)
Projet Freestyle → *Execute Windows batch command* → **Build Now** → lire **Console Output** :
```cmd
whoami
whoami /all
```
La sortie renvoyée dans la Console Output = **compte de domaine** (format `dcorp\<user>`).
→ Voir [[Cheatsheet - Privilege escalation]] §"Abuse Jenkins" étape 3-4.

### Etape 2 - Voie Script Console (Groovy) si admin
```groovy
println "cmd /c whoami".execute().text
```
Le job tourne sous le même process Jenkins → même identité.

### Etape 3 (bonus) - Confirmer les droits pour la suite
```cmd
powershell -c "whoami /groups"
powershell -c "Get-LocalGroupMember -Group Administrators"
```
Le compte doit apparaître **High Integrity** + membre `Administrators` local → prêt pour dump LSASS / OverPass-the-Hash (LO7).

## Reponse

| Element | Valeur |
|---------|--------|
| User domaine service Jenkins | dcorp\ciadmin |

## Pieges et OPSEC
- Le build step est **batch (cmd)**, pas PowerShell → `Invoke-Command is not recognized` = préfixer `powershell -c "..."` ou `powershell -enc <b64>`.
- Ne pas se bloquer à monter un reverse shell : la **Console Output** = ton shell. Reverse shell utile seulement pour l interactif (LO7).
- Traces Jenkins : `builds/<n>/log`, `audit-trail` plugin si activé. Chaque build laisse un `Console Output` archivé côté serveur.
- Le compte est admin **local** sur `dcorp-ci` mais session Jenkins tourne parfois **sans** le token élevé → tester `whoami /priv` avant de dumper LSASS.

## Refs
- [[Cheatsheet - Privilege escalation]]
- [[Cheatsheet - Post-exploitation]]
- [[Cheatsheet - Lateral movement]]
- [[LO05 - Privesc locale et Jenkins]]
- [[LO07 - Session DA et escalade]]
- [[Flags - Index]] · [[Journal - Index]]
