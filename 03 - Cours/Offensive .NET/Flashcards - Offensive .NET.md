# Flashcards - Offensive .NET

> [!summary] Comment réviser
> Réponds de tête, puis déplie. Cours : [[Offensive .NET]].

> [!question]- Quel est le lien entre PowerShell et .NET, et pourquoi ça compte ?
> PowerShell est bâti sur .NET (`System.Management.Automation.dll`). Tout ce que fait PowerShell peut se faire en C# compilé — mais en sortant du pipeline PowerShell, qui est le plus surveillé (AMSI, logging, CLM).

> [!question]- Pourquoi basculer d'un script PowerShell vers un outil .NET compilé ?
> PowerShell est durci (AMSI, Script Block Logging, Constrained Language Mode, transcription). Un assembly .NET chargé en mémoire contourne ce pipeline : pas de script visible, pas d'écriture disque.

> [!question]- Quels sont les trois défis de l'offensive .NET ?
> 1) Les contre-mesures de détection (signature AV + AMSI/ETW à l'exécution). 2) La livraison du payload sans fichier détectable. 3) La journalisation à éviter (ETW, logs .NET).

> [!question]- Comment fonctionne l'exécution en mémoire, et que fait le loader ?
> On charge l'assembly par réflexion (`Assembly.Load`) sans l'écrire sur disque. Un loader (Loader.exe / NetLoader) prend l'assembly depuis un fichier ou une URL, l'exécute en mémoire et patche AMSI + ETW au passage.

> [!question]- Pourquoi charger en mémoire change-t-il la donne face à l'AV ?
> Un binaire sur disque est scanné avant exécution ; chargé en mémoire par réflexion, il échappe au scan à l'écriture. Restent la signature en mémoire (→ obfuscation) et l'inspection runtime (→ patch AMSI/ETW).

> [!question]- À quoi servent DefenderCheck et AmsiTrigger ?
> Ils localisent précisément la chaîne ou les octets d'un binaire qui déclenchent la détection Defender/AMSI, pour savoir exactement quoi masquer.

> [!question]- Différence entre obfuscation du code source et du binaire compilé ?
> Source (ex. Codecepticon) : on renomme symboles et chaînes avant compilation, quand on a le code. Binaire (ex. ConfuserEx) : packing/renommage après compilation, quand on n'a que l'exécutable.

> [!question]- Que neutralisent respectivement le patch AMSI, le patch ETW et les syscalls directs ?
> Patch AMSI : l'inspection du code par l'AV juste avant exécution. Patch ETW : la télémétrie .NET qui alimente les EDR. Syscalls directs : les hooks EDR posés sur les API Windows en espace utilisateur.

## Voir aussi
- [[Offensive .NET]]
- [[Cours - MOC]]
