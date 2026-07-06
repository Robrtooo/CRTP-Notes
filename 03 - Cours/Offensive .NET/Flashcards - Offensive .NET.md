# Flashcards - Offensive .NET

> [!summary] Révision active
> Cartes au format Spaced Repetition — déck `#flashcards/offensive-net`. Ouvre la commande « Spaced Repetition: Review flashcards » pour réviser la file du jour. Cours : [[Offensive .NET]].

#flashcards/offensive-net

**Quel est le lien entre PowerShell et .NET, et pourquoi ça compte ?**
?
PowerShell est bâti sur .NET (`System.Management.Automation.dll`). Tout ce que fait PowerShell peut se faire en C# compilé — mais en sortant du pipeline PowerShell, qui est le plus surveillé (AMSI, logging, CLM).

**Pourquoi basculer d'un script PowerShell vers un outil .NET compilé ?**
?
PowerShell est durci (AMSI, Script Block Logging, Constrained Language Mode, transcription). Un assembly .NET chargé en mémoire contourne ce pipeline : pas de script visible, pas d'écriture disque.

**Quels sont les trois défis de l'offensive .NET ?**
?
1) Les contre-mesures de détection (signature AV + AMSI/ETW à l'exécution). 2) La livraison du payload sans fichier détectable. 3) La journalisation à éviter (ETW, logs .NET).

**Comment fonctionne l'exécution en mémoire, et que fait le loader ?**
?
On charge l'assembly par réflexion (`Assembly.Load`) sans l'écrire sur disque. Un loader (Loader.exe / NetLoader) prend l'assembly depuis un fichier ou une URL, l'exécute en mémoire et patche AMSI + ETW au passage.

**Pourquoi charger en mémoire change-t-il la donne face à l'AV ?**
?
Un binaire sur disque est scanné avant exécution ; chargé en mémoire par réflexion, il échappe au scan à l'écriture. Restent la signature en mémoire (→ obfuscation) et l'inspection runtime (→ patch AMSI/ETW).

**À quoi servent DefenderCheck et AmsiTrigger ?**
?
Ils localisent précisément la chaîne ou les octets d'un binaire qui déclenchent la détection Defender/AMSI, pour savoir exactement quoi masquer.

**Différence entre obfuscation du code source et du binaire compilé ?**
?
Source (ex. Codecepticon) : on renomme symboles et chaînes avant compilation, quand on a le code. Binaire (ex. ConfuserEx) : packing/renommage après compilation, quand on n'a que l'exécutable.

**Que neutralisent respectivement le patch AMSI, le patch ETW et les syscalls directs ?**
?
Patch AMSI : l'inspection du code par l'AV juste avant exécution. Patch ETW : la télémétrie .NET qui alimente les EDR. Syscalls directs : les hooks EDR posés sur les API Windows en espace utilisateur.

## Voir aussi
- [[Offensive .NET]]
- [[Cours - MOC]]
