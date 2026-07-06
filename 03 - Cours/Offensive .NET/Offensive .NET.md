# Offensive .NET

> [!summary] Objectif
> Comprendre pourquoi et comment l'offensive bascule de PowerShell vers des outils .NET compilés exécutés en mémoire, et quels obstacles (détection, livraison, journalisation) cette bascule cherche à contourner. Notes brutes : [[1. General Notes]], [[AV BYPASS]]. Catalogue d'outils : [[Offensiv .NET]], [[Tools list]]. Révision : [[Flashcards - Offensive .NET]].

## Ce qu'est .NET, et son lien avec PowerShell

.NET est l'environnement d'exécution de Microsoft : une machine virtuelle (le CLR), des langages (C#, VB.NET) et une immense bibliothèque. Point clé pour nous : **PowerShell est lui-même bâti sur .NET** (`System.Management.Automation.dll`). Tout ce que fait PowerShell, on peut le faire en C# compilé — mais en sortant du pipeline PowerShell, qui est justement le plus surveillé.

## Pourquoi basculer vers le .NET offensif

PowerShell est aujourd'hui fortement durci : AMSI inspecte le code avant exécution, le Script Block Logging enregistre tout, le Constrained Language Mode bride les cmdlets, la transcription journalise les sessions ([[Cheatsheet - PowerShell]]). Un outil .NET compilé (Rubeus, SafetyKatz, Certify, SharpHound…) contourne ce pipeline : on charge un **assembly** directement en mémoire et on l'exécute, sans script PowerShell visible et sans toucher au disque.

## Les trois défis de l'offensive .NET

1. **Contre-mesures de détection** — l'antivirus (signature statique) et AMSI/ETW (inspection à l'exécution) reconnaissent les outils connus.
2. **Livraison du payload** — faire arriver et charger l'assembly sur la cible sans écrire un fichier détectable.
3. **Journalisation** — éviter de laisser des traces exploitables (ETW, logs .NET).

Chaque technique d'évasion répond à l'un de ces trois défis.

## Exécution en mémoire

Le cœur de la méthode : charger un assembly .NET par réflexion (`Assembly.Load`) et l'invoquer sans jamais l'écrire sur disque. Un **loader** (Loader.exe / NetLoader) prend l'assembly depuis un fichier ou une URL, l'exécute en mémoire, et **patche AMSI et ETW** au passage pour aveugler l'inspection runtime. C'est ce même Loader que l'on retrouve devant Rubeus dans les cheatsheets d'exploitation.

> [!note] Pourquoi la mémoire change tout
> Un binaire sur disque est scanné par l'AV *avant* de s'exécuter. Chargé en mémoire par réflexion, il échappe à ce scan à l'écriture — restent la signature en mémoire (d'où l'obfuscation) et l'inspection runtime (d'où le patch AMSI/ETW).

## Contourner la détection par signature

La détection Defender la plus simple est **basée sur la signature** : elle reconnaît des chaînes ou des séquences d'octets connues dans le binaire. On la cartographie et on la casse :

- **DefenderCheck / AmsiTrigger** — localisent précisément la chaîne ou les octets qui déclenchent la détection, pour savoir quoi masquer.
- **Obfuscation du code source** — renommer symboles et chaînes avant compilation casse les signatures statiques (ex. Codecepticon, [[Offensiv .NET#^419960]]).
- **Obfuscation du binaire compilé** — packing/renommage après compilation quand on n'a pas la source (ex. ConfuserEx, [[Offensiv .NET#^oe]]).

## Contourner l'inspection à l'exécution

- **Patch AMSI** — neutralise l'interface qui soumet le code à l'antivirus juste avant exécution.
- **Patch ETW** — coupe la télémétrie .NET qui alimente les EDR.
- **Syscalls directs** — appeler les fonctions du noyau sans passer par les API Windows hookées par l'EDR (ex. CSWhispers), pour éviter les hooks en espace utilisateur.

## Le fil à retenir

PowerShell reste pratique pour l'énumération légère ; dès qu'il s'agit d'exécuter un outil offensif connu, on passe en .NET compilé, en mémoire, obfusqué, avec AMSI/ETW neutralisés. Détection, livraison, journalisation : chaque choix technique vise l'un de ces trois fronts.

## Notes associées
- [[What is PowerShell]] · [[2. Attack flow]] · [[Flashcards - Offensive .NET]]
- [[Cheatsheet - PowerShell]] · [[Cheatsheet - Post-exploitation]] · [[Tools list]]
