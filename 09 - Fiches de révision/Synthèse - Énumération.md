# Synthèse - Énumération

> [!summary] Résumé
> Fiche courte pour organiser une énumération Active Directory en lab sans perdre le fil entre réseau, annuaire et relations d'accès.

> [!warning] Cadre d'utilisation
> À utiliser uniquement dans un environnement de lab autorisé, dans le cadre de l'apprentissage et de la préparation CRTP.

## Ordre logique

1. Identifier le domaine, le DNS et les contrôleurs de domaine.
2. Vérifier les services exposés utiles : LDAP, SMB, Kerberos, DNS.
3. Recueillir les objets importants : utilisateurs, groupes, machines, partages.
4. Cartographier les relations avec BloodHound ou des outils équivalents.
5. Relier les résultats aux scénarios du lab.

## Outils souvent utilisés

- `nmap`
- `ldapsearch`
- `smbclient`
- `Get-ADUser` / `Get-DomainUser`
- `SharpHound` / `BloodHound`

## Notes associées

- [[Enumeration]]
- [[CheetSheet]]
- [[Cheatsheet - Reconnaissance AD]]
- [[Cheatsheet - BloodHound SharpHound]]

