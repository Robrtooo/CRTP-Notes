---
flag: 7
machine: dcorp-ci
statut: validé
technique: Jenkins auth (weak creds / signup)
LO: [[LO05 - Privesc locale et Jenkins]]
---

# F07 - User Jenkins console web

> [!summary] Objectif
> Trouver le nom d utilisateur utilisé pour s authentifier sur la console web Jenkins (`dcorp-ci:8080`).

## Contexte
- **Etape attack flow**: après repérage d un port applicatif inattendu (8080) sur `dcorp-ci`, entrée par l app web pour RCE.
- **Ce qu on a**: shell PowerShell student + navigateur sur la VM.
- **Ce qu on cherche**: le **username** validé côté Jenkins (réponse du flag). Sert de porte d entrée à la Script Console / Build step qui exécuteront du code sous le service account (F08).

## Solution

### Etape 1 - Localiser Jenkins
```powershell
Get-DomainComputer -Identity dcorp-ci | select dnshostname,ipv4address
```
Alternative si le module n est pas chargé : `ping dcorp-ci` renvoie l IP.
→ Voir [[Cheatsheet - Privilege escalation]] §"Abuse Jenkins".

### Etape 2 - S authentifier sur la console web
Naviguer vers `http://<dcorp-ci>:8080`. Deux voies rapides :
```text
# Voie A: pas de politique de mdp -> username = password
login: <name>   password: <name>

# Voie B: page autorise "Sign up" -> créer un compte
```
Le **username** avec lequel la session s ouvre = **F07**.

### Etape 3 - Confirmer l accès
Depuis la console → tester `http://<dcorp-ci>:8080/script` (Script Console admin) ou un job Freestyle avec droit *Configure*. Si l un des deux répond, l identité est utilisable pour la suite (F08).

## Reponse

| Element | Valeur |
|---------|--------|
| User console web Jenkins | `builduser` (mdp = username → auth "sign in as guest" possible aussi) |

## Pieges et OPSEC
- Ne pas confondre le user de connexion (F07) et le **service account** qui exécute le job (F08 = `dcorp\...`).
- La Script Console (Groovy) demande souvent des droits admin Jenkins → si refusée, passer par un job Freestyle *Execute Windows batch command*.
- Tentatives de bruteforce = 401/403 loggés côté Jenkins (`access.log`). Préférer signup ou creds par défaut.

## Refs
- [[Cheatsheet - Privilege escalation]]
- [[Cheatsheet - Lateral movement]]
- [[LO05 - Privesc locale et Jenkins]]
- [[Flags - Index]] · [[Journal - Index]]
