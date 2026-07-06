# Sliver - Index

> [!summary] Objectif
> Rejouer toute la kill chain CRTP avec le C2 **Sliver** au lieu de PowerShell direct : exécution en mémoire, injection de processus, bypass des défenses du lab. Ce dossier est autonome — même traitement que le vault principal (cheatsheets, cours, flashcards, carte d'attaque). Source : *CRTP with Sliver* (LabManual_Sliver).

> [!tip] Par où commencer
> **➜ [[Sliver - Attack map.canvas|Carte d'attaque Sliver]]** (cliquable) et le cours **[[Qu'est-ce que Sliver]]**. La logique offensive AD reste la même — voir [[2. Attack flow]] côté vault principal ; ici on apprend *comment l'opérer via Sliver*.

## Cours

- [[Qu'est-ce que Sliver]] — C2, beacon vs session, implants, listeners, armory, OPSEC
- [[Workflow Sliver]] — fork & run vs self-inject, PPID/migrate, execute-assembly, livraison
- Flashcards :
    - [[Flashcards - Sliver]]
    - [[Flashcards - Workflow Sliver]]

## Cheatsheets

Master : **[[Sliver - Toutes les commandes]]**.

- [[Sliver - Setup et beacons]] — serveur, listeners, generate, delivery, sessions/beacons, migrate
- [[Sliver - Exécution et évasion]] — execute-assembly (`-i -M -E`), inline-execute, PEzor, armory, bypass des logs/AMSI
- [[Sliver - Énumération AD]] — StandIn, ADSearch, ADCollector, SharpHound
- [[Sliver - Kerberos et tickets]] — Rubeus (asktgt, kerberoast, s4u), Golden/Silver/Diamond, DCSync
- [[Sliver - Delegation et RBCD]] — unconstrained + SpoolSample, constrained S4U, RBCD
- [[Sliver - Privesc et lateral]] — SharpUp/Seatbelt, scshell/remote-sc, sa-*, WMI/winrs, Jenkins
- [[Sliver - AD CS, SQL et persistance]] — Certify (ESC1/ESC3), SharpSQL/PowerUpSQL, DSRM, LSASS

## Repères lab

- Foothold : `dcorp-stdX` — `172.16.100.X`, user `dcorp\studentX`.
- Serveur Sliver : WSL Ubuntu sur le foothold (`/mnt/c/AD/Tools/Sliver`).
- Outils : `C:\AD\Tools\Sliver`, implants générés dans `…\Sliver\Implants`.

## Correspondance avec le vault principal

Les concepts (Kerberos, délégation, AD CS…) sont expliqués côté cours principal — ce dossier ne réexplique pas la théorie, il donne l'exécution Sliver. Voir [[Attaques AD]], [[Kerberos]], [[2. Attack flow]].
