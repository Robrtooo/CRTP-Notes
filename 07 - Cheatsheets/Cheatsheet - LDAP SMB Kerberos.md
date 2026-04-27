# Cheatsheet - LDAP SMB Kerberos

> [!summary] Objectif
> Centraliser les requêtes simples utiles pour interroger LDAP, tester SMB et confirmer le fonctionnement de Kerberos dans un lab Active Directory.

> [!warning] Cadre d'utilisation
> À utiliser uniquement dans un environnement de lab autorisé, dans le cadre de l'apprentissage et de la préparation CRTP.

## Quand utiliser cette fiche

- Après la reconnaissance initiale du domaine.
- Quand il faut comparer plusieurs sources d'information AD.

## Prérequis

- Un compte du lab quand l'authentification est nécessaire.
- Les valeurs `<DOMAIN>`, `<DC_IP>`, `<USER>` et éventuellement `<PASSWORD>`.

## Commandes

### Objectif

Comparer PowerView et le module AD sur les objets du domaine.

### Commande

```powershell
. C:\AD\Tools\PowerView.ps1
Get-DomainUser
Get-DomainComputer | Select-Object Name
Get-ADUser -Filter * -Properties *
Get-ADComputer -Filter * | Select-Object Name
```

### Explication

Cette séquence reprend les deux familles d'outils utilisées dans les supports CRTP pour valider les mêmes objets côté annuaire.

### Contexte d’utilisation

Pratique sur la VM étudiante pour voir rapidement quelles sorties sont les plus utiles à garder en notes.

### Précautions

Éviter `-Properties *` si un jeu d'attributs plus ciblé suffit.

### Objectif

Tester les partages SMB visibles.

### Commande

```bash
smbclient -L //<DC_IP>/ -U <USER>
```

### Explication

Permet de voir les partages exposés et de confirmer si l'authentification SMB fonctionne.

### Contexte d’utilisation

Utile quand on veut lier l'énumération du domaine aux ressources réellement accessibles.

### Précautions

Ne parcourir que les partages du lab explicitement autorisés.

### Objectif

Effectuer une requête LDAP simple depuis Linux.

### Commande

```bash
ldapsearch -x -H ldap://<DC_IP> -b "DC=<DOMAIN>,DC=local"
```

### Explication

Effectue une requête LDAP simple pour vérifier la structure du domaine et la visibilité des objets depuis un poste Linux.

### Contexte d’utilisation

Utile pour comparer une approche Linux avec l'énumération PowerShell du support.

### Précautions

Adapter la base DN au domaine du lab.

### Objectif

Valider le domaine côté PowerShell.

### Commande

```powershell
Get-ADDomain
Get-ADDomainController
```

### Explication

Les cmdlets du module Active Directory donnent rapidement le nom du domaine, son SID et ses contrôleurs.

### Contexte d’utilisation

Très utile sur un hôte Windows de lab joint au domaine.

### Précautions

Le module Active Directory doit être disponible sur l'hôte utilisé.

### Notes associées

- [[CheetSheet]]
- [[Enumeration]]
- [[What is PowerShell]]
- [[Components of AD]]
- [[Cheatsheet - Reconnaissance AD]]
