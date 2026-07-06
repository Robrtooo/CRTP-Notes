# Synthèse - Monitoring et détections

> [!summary] Résumé
> Défenses à connaître pour l'OPSEC. Cours : [[What is PowerShell]], [[Offensive .NET]]. Fiche : [[Cheatsheet - Évasion et OPSEC]].

## Contrôles à distinguer

- **Transcription** + **Script Block Logging** : journalisent le code PowerShell exécuté.
- **AMSI** : soumet script/assembly à l'AV *avant* exécution.
- **ETW** : télémétrie .NET/CLR qui alimente les EDR.
- **Constrained Language Mode (CLM)** : bride les cmdlets et l'accès aux objets .NET.
- **AppLocker** : contrôle quelles binaires/scripts peuvent s'exécuter.
- **MDE / MDI** : Defender for Endpoint (hôte) / for Identity (annuaire).

## Réflexe

La session distante n'est pas équivalente à la session locale (CLM, WinRM). Avant de conclure à un échec de commande, vérifier le contexte de défense. Contournements : [[Cheatsheet - Évasion et OPSEC]].

## Notes associées

- [[Cheatsheet - Monitoring et détections]] · [[Cheatsheet - Évasion et OPSEC]] · [[What is PowerShell]] · [[Offensive .NET]]
