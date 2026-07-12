---
flag: 8
machine: dcorp-ci
statut: validé
---
# F08 · User de domaine du service Jenkins

> [!question] LO5 — dcorp-ci
> Domain user used for running Jenkins service on dcorp-ci

## Commande qui marche
Le build step Jenkins (`Execute Windows batch command`) tourne **en tant que le compte de service** → un `whoami` dedans révèle le compte de domaine (sortie = Console Output, pas besoin de shell) :
```cmd
whoami
```
→ **`dcorp\ciadmin`** = compte de domaine qui exécute le service Jenkins (admin High sur dcorp-ci).

## Fiches liées
[[Cheatsheet - Privilege escalation]] · [[LO05 - Privesc locale et Jenkins]] · [[LO07 - Session DA et escalade]]

[[Flags - Index]] · [[Journal - Index]]
