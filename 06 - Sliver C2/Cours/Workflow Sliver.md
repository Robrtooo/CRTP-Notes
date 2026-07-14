# Workflow Sliver

> [!summary] Objectif
> Comprendre le modèle opératoire de bout en bout : comment on passe du foothold à l'exécution d'outils en mémoire, et surtout **comment choisir sa technique d'injection** selon la furtivité. C'est le savoir-faire qui rend les cheatsheets utilisables intelligemment. Révision : [[Flashcards - Workflow Sliver]].

## La boucle opératoire

1. **Serveur + listener** (WSL, `https`).
2. **Générer** un implant shellcode obfusqué (`generate -e -f shellcode`).
3. **Livrer** : webserver + dropper NtAPI (BinLoader) → injection → **beacon**.
4. **Stabiliser** : `migrate` vers un processus durable (`explorer.exe`).
5. **Opérer** : `execute-assembly` / BOF pour chaque technique.
6. **Pivoter** et recommencer sur la machine suivante.

## PPID spoofing : se fondre dans l'arbre des processus

Les défenses regardent les relations **parent/enfant**. Un `WINWORD.exe` qui lance `cmd.exe` est suspect. Le **PPID spoofing** permet de choisir un parent crédible : on imite des relations légitimes (`svchost.exe` → `RuntimeBroker.exe` / `sihost.exe` / `taskhostw.exe`). D'où l'étape `migrate` vers un processus stable, qui devient l'hôte de nos futures injections.

## Le cœur : execute-assembly

`execute-assembly` exécute un assembly .NET **en mémoire**, sans l'écrire sur disque ni lancer PowerShell. Deux modes, deux compromis :

### Fork & run (`-p <processus>`)

Spawne un **processus sacrificiel** sous un parent usurpé, y injecte le code, l'exécute, puis le tue. Avantages : isolation (un crash n'affecte pas le beacon), parent crédible pour l'OPSEC. Inconvénient : création de processus = un événement observable. À privilégier pour les actions longues ou instables.

### Self-inject (`-i -M -E`)

Injecte dans le **processus du beacon lui-même** : aucun processus enfant créé (moins d'IOC de spawn), et accès aux **bypass AMSI (`-M`) et ETW (`-E`)** intégrés. Inconvénient : un assembly qui plante peut tuer le beacon. À privilégier pour les actions ponctuelles et furtives (Rubeus, Certify).

### inline-execute-assembly / BOF

Pour la self-injection pure, sans fork & run. Les **BOF** (`sa-*`, `remote-sc-*`) sont encore plus légers : de petits objets compilés chargés dans le beacon, idéaux pour l'énumération de services/partages ou l'exécution par service, sans spawner d'assembly complet.

> [!note] Règle de décision
> **Furtif + ponctuel** → self-inject `-i -M -E`. **Long / instable / besoin d'un parent crédible** → fork & run `-p <proc>`. **Tâche système ciblée** → BOF `sa-*` / `remote-sc-*`. Vérifier avec Process Hacker (présence d'`amsi.dll`, PID du parent).

## PEzor : quand l'outil doit voyager

Certains outils signés doivent être **délivrés et exécutés à distance** (ex. sur un DC via un service). **PEzor** empaquette un assembly en shellcode packé (anti-sandbox, unhook), qu'on injecte comme le beacon initial (via `remote-sc-*` ou BinLoader). C'est le pont entre « exécuter localement en mémoire » et « exécuter sur une autre machine sans y poser de binaire détectable ».

## Pivots et propagation

Une fois admin sur une machine, on la transforme en relais : **pivots TCP / named pipe** (`pivots tcp --lport 8081`) font transiter le C2 des machines internes (sans accès Internet direct) par le foothold. Combiné à `scshell` / `winrs` / WMI, on se déplace de proche en proche jusqu'au DC.

## Relier à la kill chain

Ce workflow est le « comment » ; le « quoi/quand » reste [[Attack flow]] : chaque état (compte → droit ACL → admin local → hash → DA → EA) se joue avec `execute-assembly` + l'outil adapté. La carte [[Sliver - Attack map.canvas|Attack map Sliver]] fait le lien visuel.

## Notes associées
- [[Qu'est-ce que Sliver]]
- [[Flashcards - Workflow Sliver]]
- [[w. Sliver C2/Sliver - Exécution et évasion]]
- [[Attack flow]]
- [[Sliver - Index]]
