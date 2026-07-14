# Writeups CRTP

> Writeups d'exploitation du lab CRTP. Deux niveaux de granularité.

## Flags F01-F40 — 1 writeup complet par flag

Chaque fichier `F$XX - Titre.md` = writeup autonome avec contexte, solution étape par étape, commandes prêtes à copier, placeholder `A REMPLIR DEPUIS LAB` pour les valeurs spécifiques (hashes, SPN, templates).

- **[[Flags - Index|📋 Index des Flags]]** — tableau de tous les flags avec statut
- Accès direct par technique :

| Domaine | Flags |
|---------|-------|
| Enum & ACL | [[F01 - SID membre du groupe Enterprise Admins\|F01]] · [[F02 - Droits AD de RDPUsers sur ControlxUser\|F02]] · [[F03 - GPO appliquée sur StudentMachines OU\|F03]] · [[F04 - Sens du trust dollarcorp - eurocorp\|F04]] |
| Privesc & Jenkins | [[F05 - Service abusé pour privesc locale\|F05]] · [[F06 - Script hunt admin via PS Remoting\|F06]] · [[F07 - User Jenkins console web\|F07]] · [[F08 - User de domaine du service Jenkins\|F08]] |
| GPO mod & svcadmin | [[F09 - Attribut Group Policy modifié\|F09]] · [[F10 - Process utilisant svcadmin\|F10]] · [[F11 - NTLM de svcadmin\|F11]] · [[F12 - Source des creds tâches planifiées\|F12]] |
| NTLM & DCSync | [[F13 - NTLM-clair de srvadmin\|F13]] · [[F14 - NTLM de websvc\|F14]] · [[F15 - NTLM de appadmin\|F15]] · [[F16 - NTLM de krbtgt\|F16]] · [[F17 - NTLM du Domain Admin (Administrator)\|F17]] |
| Silver / Diamond / DSRM | [[F18 - Service pour Silver Ticket (winrs-PSRemoting)\|F18]] · [[F19 - Compte source du Diamond Ticket\|F19]] · [[F20 - Clé registre logon DSRM\|F20]] |
| DCSync SD / Kerberoast | [[F21 - Attaque avec droits de réplication\|F21]] · [[F22 - SDDL WMI root-cimv2 (studentx = BA)\|F22]] · [[F23 - SPN kerberoasté (TGS demandé)\|F23]] |
| Delegation | [[F24 - User domaine local admin dcorp-appsrv\|F24]] · [[F25 - Creds compromis via printer bug\|F25]] · [[F26 - Valeur msds-allowedtodelegateto adminsrv\|F26]] · [[F27 - Service alternatif via constrained deleg\|F27]] · [[F28 - Machine où ciadmin configure RBCD\|F28]] |
| Trusts cross-forest | [[F29 - SID history injecté (Enterprise Admins)\|F29]] · [[F30 - NTLM krbtgt de moneycorp.local\|F30]] · [[F31 - Service TGS demandé sur eurocorp-dc\|F31]] · [[F32 - Contenu de secret.txt (eurocorp-dc)\|F32]] |
| AD CS | [[F33 - Template ADCS ENROLLEE_SUPPLIES_SUBJECT\|F33]] · [[F34 - Template ADCS Certificate Request Agent\|F34]] · [[F35 - Attribut CA autorisant les SAN\|F35]] · [[F36 - Groupe droits enroll sur CA-Integration\|F36]] |
| MSSQL | [[F37 - Premier SQL Server lié à dcorp-mssql\|F37]] · [[F38 - User SQL du lien sql1 - mgmt\|F38]] · [[F39 - Privilèges SQL sur eu-sql\|F39]] · [[F40 - Privilèges OS sur eu-sql\|F40]] |

## Learning Objectives LO01-LO23 — techniques par thème

- **[[Journal - Index|📚 Index des LOs]]** — tableau de progression
- Fichiers dans `Learning/` — 1 fiche technique par LO du lab.

## Comment naviguer

- Depuis une **cheatsheet** → link vers le flag correspondant (contexte concret)
- Depuis un **flag** → link vers cheatsheet (variantes commandes) + LO (technique fond)
- Depuis un **cours** → link vers flags qui illustrent la technique
