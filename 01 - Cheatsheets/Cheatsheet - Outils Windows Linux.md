# Cheatsheet - Outils Windows Linux

> [!summary] Objectif
> Réflexe rapide selon la machine d'attaque disponible dans le lab (Windows joint au domaine vs Linux).

### Depuis Linux

```bash
nmap -sV -Pn <TARGET>
```

### Depuis Windows (joint au domaine)

```powershell
Get-ADUser -Filter * -Properties *
Get-ADComputer -Filter *
```

Réduire les propriétés demandées (`-Properties <attribut>`) dès qu'une sortie ciblée suffit plutôt que `*`.

**Link** — [[What is PowerShell]] · [[Enumeration]] · [[CheetSheet]] · [[Cheatsheets - MOC]]
