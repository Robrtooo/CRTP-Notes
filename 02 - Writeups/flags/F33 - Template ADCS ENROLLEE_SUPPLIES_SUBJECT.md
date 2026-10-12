---
flag: 33
machine: dcorp-dc
statut: à faire
technique: ADCS-ESC1
LO: "[[LO21 - AD CS (ESC1 et ESC3)]]"
---

# F33 · Template ADCS ENROLLEE_SUPPLIES_SUBJECT

> [!summary] Objectif
> Identifier le template AD CS marqué `ENROLLEE_SUPPLIES_SUBJECT` — vecteur ESC1 (SAN arbitraire → cert au nom d'un DA).

## Contexte
- **Etape attack flow**: post-compromission d'un simple Domain User, énumération AD CS avant escalade DA/EA via PKINIT.
- **Ce qu on a**: session sur `dcorp-studentx` avec un compte Domain Users (pas de privilège local ni AD particulier).
- **Ce qu on cherche**: le **nom exact du template** (attribut `Name` / `msPKI-Cert-Template-OID`) qui expose le flag `CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT` dans `msPKI-Certificate-Name-Flag`.

## Solution

### Etape 1: Enumerer CA et templates
```powershell
C:\AD\Tools\Certify.exe cas
C:\AD\Tools\Certify.exe find
```
→ Voir [[Cheatsheet - AD CS et certificats]] pour la sortie type.

### Etape 2: Filtrer sur ENROLLEE_SUPPLIES_SUBJECT
```powershell
C:\AD\Tools\Certify.exe find /enrolleeSuppliesSubject
```
Chercher dans la sortie : `msPKI-Certificate-Name-Flag: ENROLLEE_SUPPLIES_SUBJECT` + `Enrollment Rights` ouvert à `Domain Users` + EKU `Client Authentication` (les 3 = ESC1 complet).

### Etape 3 (optionnel): Confirmation vulnérable
```powershell
C:\AD\Tools\Certify.exe find /vulnerable
```

## Reponse

| Element | Valeur |
|---------|--------|
| Nom du template (ESC1) | <A REMPLIR DEPUIS LAB> |
| CA hébergeant le template | <A REMPLIR DEPUIS LAB> |

## Pieges et OPSEC
- Ne pas confondre `ENROLLEE_SUPPLIES_SUBJECT` (flag template = ESC1) avec `EDITF_ATTRIBUTESUBJECTALTNAME2` (flag CA = ESC6 → F35). Deux niveaux différents.
- Un template avec le flag mais **sans** EKU Client Auth n'est pas exploitable en ESC1.
- Trace : Event ID **4886/4887** sur le CA (Certificate Services Request) + Event **4768** sur le DC (TGT via PKINIT anormal).

## Refs
- [[Cheatsheet - AD CS et certificats]]
- [[Synthèse - AD CS]]
- [[LO21 - AD CS (ESC1 et ESC3)]]
- [[Attaques AD]]
