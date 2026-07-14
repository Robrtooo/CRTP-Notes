---
lo: 5
statut: en cours
---
# LO5 - Privesc locale et Jenkins

> [!info] Objectif
> Énumérer un service vulnérable (SharpUp/Seatbelt) → élever en admin local. Identifier où studentX est admin local. Abuser Jenkins pour un accès admin sur dcorp-ci.

## Fiches liées

- [[Cheatsheet - Privilege escalation]]
- [[Cheatsheet - LAPS]]
- [[LO07 - Session DA et escalade]]

## Abus Jenkins → RCE `ciadmin` sur dcorp-ci

Jenkins sur `dcorp-ci:8080`. Un **build step** `Execute Windows batch command` s'exécute en tant que **`dcorp\ciadmin`** (admin **High** sur dcorp-ci) → RCE directe.

> [!tip] RCE ≠ reverse shell
> Le build step renvoie son **stdout dans le Console Output** = ton shell. Pas besoin de reverse shell ni de shell interactif pour énumérer/pivoter. Ne te bloque pas à monter un listener.

> [!warning] C'est du `cmd`, pas du PowerShell
> `'Invoke-Command' is not recognized` = batch. Préfixe **`powershell -c "..."`** (ou `powershell -enc <b64>` pour éviter les guillemets).

```cmd
whoami
powershell -c "whoami /groups"
```

## Suite = LO7

ciadmin est **admin local dérivé sur dcorp-mgmt** (session DA `svcadmin`). Toute la chaîne DA (pivot outbound, Defender off, dump comsvcs, exfil Jenkins userContent, OverPass-the-Hash) est détaillée dans **[[LO07 - Session DA et escalade]]**.

## Piège / à retenir

> [!failure]- Pod verrouillé : student VM non-admin + FW inbound fermé
> Impossible d'héberger HFS ou de recevoir un reverse shell (le chemin standard du manual). Tout se fait **outbound / Console Output**. Détails et fixes dans [[LO07 - Session DA et escalade]].

## Voir aussi
[[Journal - Index]] · [[Attack flow]]
