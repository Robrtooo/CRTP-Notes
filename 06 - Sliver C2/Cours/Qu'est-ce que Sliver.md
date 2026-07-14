# Qu'est-ce que Sliver

> [!summary] Objectif
> Comprendre ce qu'est un C2 et comment Sliver structure une opération offensive : implants, listeners, beacon vs session, armory, et la philosophie OPSEC du lab (ne jamais lancer PowerShell directement, tout exécuter en mémoire). C'est la base conceptuelle avant les cheatsheets. Révision : [[Flashcards - Sliver]].

## Un C2, c'est quoi

Un **Command & Control** est l'infrastructure qui relie l'opérateur à ses accès sur le réseau cible. Deux moitiés : un **serveur** (chez l'attaquant) qui reçoit les callbacks et envoie les ordres, et des **implants** (sur les machines compromises) qui rappellent le serveur et exécutent les tâches. Sliver (Bishop Fox) est un C2 open-source en Go, multi-OS, multi-protocoles, scriptable — pensé pour la simulation d'adversaire.

## Listeners et canaux de sortie

Le serveur ouvre des **listeners** qui attendent les callbacks. Sliver supporte plusieurs canaux : **mTLS** et **HTTPS** (chiffrés, se fondent dans le trafic web), **DNS** (traverse les réseaux verrouillés), **WireGuard**. Dans le lab on utilise HTTPS sur 443. Le choix du canal est un choix OPSEC : plus il ressemble au trafic légitime, plus il est discret.

## Implants : beacon vs session

Un **implant** est le code qui tourne sur la cible. Deux modes de communication :

- **Beacon** — rappelle le serveur à intervalles réguliers (avec du jitter). Asynchrone, discret : le trafic ressemble à des requêtes périodiques anodines. C'est le mode par défaut pour rester furtif.
- **Session** — connexion interactive, temps réel. Pratique pour travailler vite, mais plus bruyante (trafic continu).

On bascule de l'un à l'autre selon le besoin ; les commandes de tasking sont identiques. Un implant se génère avec `generate` sous plusieurs formats (exécutable, DLL, **shellcode**). Dans le lab, on produit du **shellcode** obfusqué injecté par un dropper NtAPI — rien n'est écrit sur disque en clair.

## Obfuscation et livraison

`generate -e` active l'obfuscation à la compilation (renommage de symboles + encodage shikata-ga-nai). La livraison passe par un webserver + un **dropper** (BinLoader/NtDropper) qui télécharge le shellcode et l'injecte dans un processus. Résultat : un beacon en mémoire, sans binaire offensif posé sur le disque, donc invisible au scan à l'écriture.

## L'armory

Sliver embarque des modules natifs, mais surtout une **armory** : un catalogue d'outils téléchargeables comme commandes intégrées. Deux familles :

- **Alias .NET** (SharpUp, Rubeus, Certify, SharpHound…) — des assemblies exécutés en mémoire via `execute-assembly`.
- **BOF / COFF** (`sa-*`, `remote-sc-*`…) — de petits objets compilés chargés directement dans le beacon, sans spawner de processus. Plus légers et plus discrets pour des tâches ciblées (énumération de services, partages, tâches planifiées).

`armory install <nom>` suffit à ajouter un outil au workflow.

## La philosophie OPSEC du lab

Le fil rouge du parcours Sliver : **ne pas exécuter PowerShell directement** (c'est le plus surveillé), tout faire **en mémoire**, et **bypasser les défenses** du lab :

- **Journalisation PowerShell** : PSReadline, Script Block Logging, Module Logging, transcription système.
- **AMSI / Defender** : inspection du code avant exécution.
- **ETW** : télémétrie .NET vers les EDR.

Sliver répond avec l'injection de processus, le **PPID spoofing** (se faire passer pour un enfant légitime), et les bypass AMSI/ETW intégrés à `execute-assembly` (`-M`, `-E`). Le détail est dans [[Workflow Sliver]] et [[w. Sliver C2/Sliver - Exécution et évasion]].

## Ce qui change (et ne change pas) par rapport au CRTP classique

La **logique offensive AD est identique** : énumérer → élever → se déplacer → dominer → persister (voir [[Attack flow]], [[Attaques AD]]). Ce qui change, c'est l'**outillage et l'exécution** : au lieu de scripts PowerShell (PowerView, Invoke-Mimikatz), on opère des assemblies .NET (StandIn, Rubeus, Certify) et des BOF via un C2, en mémoire, avec une hygiène OPSEC constante.

## Notes associées
- [[Workflow Sliver]]
- [[Flashcards - Sliver]]
- [[Sliver - Index]]
- [[Sliver - Setup et beacons]]
- [[Attack flow]]
- [[Attaques AD]]
