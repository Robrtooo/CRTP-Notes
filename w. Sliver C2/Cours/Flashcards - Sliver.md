# Flashcards - Sliver

> [!summary] Révision active
> Cartes au format Spaced Repetition — déck `#flashcards/sliver`. Lance « Spaced Repetition: Review flashcards ». Cours : [[Qu'est-ce que Sliver]].

#flashcards/sliver

## Concepts C2

**Qu'est-ce qu'un C2 et de quoi se compose Sliver ?**
?
Un Command & Control relie l'opérateur à ses accès : un serveur (côté attaquant) qui reçoit les callbacks et envoie les ordres, et des implants (sur les cibles) qui rappellent et exécutent les tâches. Sliver est un C2 open-source en Go, multi-OS, multi-protocoles.

**Quels canaux de sortie Sliver supporte-t-il, et pourquoi ce choix compte ?**
?
mTLS, HTTPS, DNS, WireGuard. C'est un choix OPSEC : plus le canal ressemble au trafic légitime (HTTPS/mTLS sur 443), plus il est discret. DNS traverse les réseaux verrouillés.

**Beacon vs session : différence et usage ?**
?
Le beacon rappelle le serveur à intervalles (asynchrone, discret) — mode par défaut pour la furtivité. La session est interactive/temps réel (pratique mais bruyante). Mêmes commandes de tasking, on bascule selon le besoin.

## Livraison & armory

**Comment livre-t-on l'implant dans le lab, et pourquoi est-ce discret ?**
?
On génère du shellcode obfusqué (`generate -e -f shellcode`), hébergé sur un webserver, puis injecté par un dropper NtAPI (BinLoader). Aucun binaire offensif n'est écrit sur disque → invisible au scan à l'écriture.

**Qu'est-ce que l'armory et quelles deux familles d'outils propose-t-elle ?**
?
Un catalogue d'outils installables comme commandes intégrées (`armory install`). Deux familles : alias .NET (Rubeus, Certify, SharpUp… via execute-assembly) et BOF/COFF (`sa-*`, `remote-sc-*`) chargés dans le beacon sans spawner de processus.

## OPSEC

**Quelle est la philosophie OPSEC du lab Sliver ?**
?
Ne jamais lancer PowerShell directement (le plus surveillé), tout exécuter en mémoire, et bypasser les défenses : journalisation PowerShell (PSReadline, Script Block/Module Logging, transcription), AMSI/Defender, ETW.

**Qu'est-ce qui change et ne change pas par rapport au CRTP classique ?**
?
La logique offensive AD est identique (énumérer → élever → se déplacer → dominer → persister). Ce qui change : l'outillage et l'exécution — des assemblies .NET et des BOF via un C2, en mémoire, avec OPSEC constante, au lieu de scripts PowerShell.

## Voir aussi
- [[Qu'est-ce que Sliver]]
- [[Workflow Sliver]]
- [[Sliver - Index]]
