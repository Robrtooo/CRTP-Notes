# Flashcards - PowerShell

> [!summary] Comment réviser
> Réponds de tête, puis déplie. Cours : [[What is PowerShell]].

> [!question]- PowerShell, c'est `powershell.exe` ?
> Non. PowerShell est l'assembly `System.Management.Automation.dll`. `powershell.exe` n'est qu'un des hôtes qui la charge — d'où l'intérêt d'hôtes alternatifs (ex. InviShell) pour échapper au logging.

> [!question]- Quelles sont les 4 grandes détections PowerShell de Microsoft ?
> 1) System-Wide Transcription, 2) Script Block Logging, 3) AMSI, 4) Constrained Language Mode (CLM, intégré à AppLocker/WDAC).

> [!question]- À quel moment AMSI inspecte-t-il le code, et pourquoi est-ce gênant pour un download cradle ?
> Au moment de l'exécution (runtime), y compris pour du code chargé en mémoire — donc charger un script "en mémoire pour éviter le disque" ne suffit pas : AMSI le voit quand même. D'où le besoin d'un bypass AMSI.

> [!question]- Que bride le Constrained Language Mode (CLM) ?
> Il limite les fonctionnalités du langage : pas de `Add-Type`, réflexion .NET restreinte, appels d'API bridés. À vérifier avant tout chargement de module offensif.

> [!question]- L'Execution Policy est-elle un contrôle de sécurité ?
> Non. Elle empêche seulement l'exécution accidentelle de scripts et se contourne trivialement — ce n'est pas une barrière défensive.

> [!question]- Comment InviShell évite-t-il le logging PowerShell ?
> Il hooke l'assembly .NET (pas `powershell.exe`) via la CLR Profiler API : le profiler DLL est chargé par le CLR au runtime, ce qui permet d'exécuter PS sans transcription ni script block logging.

> [!question]- À quoi servent AMSITrigger et DefenderCheck, et quelle est la différence ?
> Les deux localisent la portion *précise* de code qui déclenche la détection. AMSITrigger cible ce que déclenche **AMSI** ; DefenderCheck ce que flag **Defender**. Workflow : scanner → modifier la portion flaggée → rescanner → répéter jusqu'à non-détection.

> [!question]- Différence entre obfusquer un script PowerShell et un binaire .NET ?
> Script PowerShell → `Invoke-Obfuscation` (casse les signatures AMSI). Code source C#/.NET → `Codecepticon` (avant compilation). Binaire .NET compilé → `ConfuserEx`.

> [!question]- Pourquoi "moins de code = mieux" en évasion ?
> Moins de surface de code = moins de signatures potentielles et une analyse plus simple. Ne garder que les fonctionnalités strictement nécessaires réduit ce que les outils de détection peuvent accrocher.

> [!question]- Après avoir obfusqué/passé les outils, le travail est-il fini ?
> Non — une relecture manuelle reste souvent nécessaire (même après DefenderCheck) : revenir à la ligne/fonction signalée, comprendre si c'est une chaîne, un bloc ou une fonctionnalité, et réécrire le minimum (ex. transformer une chaîne base64 et la reconstruire au runtime).

## Voir aussi
- [[What is PowerShell]]
