# Flashcards - Workflow Sliver

> [!summary] Comment réviser
> Réponds de tête, puis déplie. Cours : [[Workflow Sliver]].

## Boucle & injection

> [!question]- Quelles sont les étapes de la boucle opératoire Sliver ?
> 1) Serveur + listener (WSL, https). 2) Générer un implant shellcode obfusqué. 3) Livrer (webserver + dropper NtAPI → beacon). 4) Stabiliser via migrate vers un processus durable. 5) Opérer avec execute-assembly / BOF. 6) Pivoter et recommencer.

> [!question]- Qu'est-ce que le PPID spoofing et pourquoi migrer vers explorer.exe ?
> Se faire passer pour l'enfant d'un parent crédible (svchost → RuntimeBroker/sihost/taskhostw) pour déjouer les détections parent/enfant. Migrer vers un processus stable comme explorer.exe donne une session résiliente et un bon parent pour les injections.

> [!question]- execute-assembly fork & run (`-p`) : mécanisme et compromis ?
> Spawne un processus sacrificiel sous un parent usurpé, y injecte et exécute le code, puis le tue. Avantage : isolation + parent crédible. Inconvénient : création de processus observable. À privilégier pour les actions longues/instables.

> [!question]- execute-assembly self-inject (`-i -M -E`) : mécanisme et compromis ?
> Injecte dans le processus du beacon lui-même : aucun processus enfant (moins d'IOC) et accès aux bypass AMSI (-M) et ETW (-E). Inconvénient : un assembly qui plante peut tuer le beacon. À privilégier pour les actions ponctuelles et furtives.

> [!question]- Quand utiliser un BOF (`sa-*`, `remote-sc-*`) plutôt qu'un assembly ?
> Pour une tâche système ciblée et légère (énumération de services/partages, exécution par service) : le BOF est chargé directement dans le beacon, sans spawner d'assembly complet — plus léger et plus discret.

## Portée & propagation

> [!question]- À quoi sert PEzor dans le workflow ?
> À empaqueter un assembly signé en shellcode packé (anti-sandbox, unhook) pour le délivrer et l'exécuter à distance (ex. sur un DC via un service), sans y poser de binaire détectable. C'est le pont entre exécution locale en mémoire et exécution sur une autre machine.

> [!question]- Comment Sliver propage le C2 sur des machines sans accès Internet ?
> Via des pivots TCP / named pipe (`pivots tcp --lport 8081`) : les machines internes font transiter leur trafic C2 par le foothold. Combiné à scshell / winrs / WMI, on se déplace de proche en proche jusqu'au DC.

## Voir aussi
- [[Workflow Sliver]]
- [[Qu'est-ce que Sliver]]
- [[Sliver - Exécution et évasion]]
