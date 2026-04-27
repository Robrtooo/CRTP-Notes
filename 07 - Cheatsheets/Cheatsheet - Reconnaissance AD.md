# Cheatsheet - Reconnaissance AD

> [!summary] Objectif
> Rassembler les commandes de base pour identifier un domaine, un contrôleur de domaine et les premiers services exposés en lab CRTP.

> [!warning] Cadre d'utilisation
> À utiliser uniquement dans un environnement de lab autorisé, dans le cadre de l'apprentissage et de la préparation CRTP.

## Quand utiliser cette fiche

- Au début d'un lab pour cartographier rapidement le domaine cible.
- Avant d'utiliser des outils AD plus spécialisés.

## Prérequis

- Accès réseau au segment du lab.
- Un hôte de départ Windows ou Linux autorisé.
- Les IP ou noms de domaine du lab, par exemple `<DC_IP>` et `<DOMAIN>`.

## Commandes

### Objectif

Identifier le domaine et le contrôleur de domaine via DNS.

### Commande

```bash
nslookup <DOMAIN>
nslookup <DC_IP>
```

### Explication

Confirme la résolution DNS et donne un premier point d'appui pour la suite de l'énumération.

### Contexte d’utilisation

Utile dès l'entrée dans le lab pour valider les éléments de base avant les requêtes LDAP, SMB ou Kerberos.

### Précautions

Rester sur les hôtes du lab autorisé et documenter les réponses obtenues.

### Objectif

Confirmer le contexte du domaine avec les commandes de base vues dans le support.

### Commande

```powershell
. C:\AD\Tools\PowerView.ps1
Get-Domain
Get-DomainSID
Get-DomainController
Get-DomainPolicyData
```

### Explication

PowerView permet de confirmer rapidement le domaine, son SID, ses contrôleurs et la politique du domaine depuis la VM du lab.

### Contexte d’utilisation

Très utile au début d'une session Windows du lab pour recouper les informations DNS avec l'annuaire.

### Précautions

Vérifier que `PowerView.ps1` est bien chargé avant d'appeler les cmdlets.

### Objectif

Identifier les ports et services pertinents d'un contrôleur de domaine.

### Commande

```bash
nmap -sV -Pn -p 53,88,135,139,389,445,464,636,3268,3269 <DC_IP>
```

### Explication

Cette commande cible les services les plus utiles pour la découverte AD : DNS, Kerberos, LDAP, SMB et Global Catalog.

### Contexte d’utilisation

À utiliser après la résolution DNS pour confirmer les services réellement accessibles dans le lab.

### Précautions

Limiter le scan aux ports utiles pour éviter du bruit inutile pendant l'apprentissage.

### Notes associées

- [[Enumeration]]
- [[CheetSheet]]
- [[Components of AD]]
- [[General Notes]]
- [[Active Directory - MOC]]
