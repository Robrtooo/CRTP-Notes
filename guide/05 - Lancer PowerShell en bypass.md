# 05 — Lancer PowerShell en bypass

## L'argument
```powershell
powershell -ep bypass
```
`-ep` = raccourci de `-ExecutionPolicy`. Valeur `Bypass` = plus aucun blocage de script.

## Combo CRTP typique
```powershell
powershell -nop -ep bypass
```
- `-nop` = `-NoProfile` → n'exécute pas le profil (plus rapide, moins de traces/hooks)
- `-ep bypass` = `-ExecutionPolicy Bypass`

## Flags utiles
| Flag | Long | Rôle |
|---|---|---|
| `-ep bypass` | `-ExecutionPolicy Bypass` | ignore l'execution policy |
| `-nop` | `-NoProfile` | pas de profil PowerShell |
| `-w hidden` | `-WindowStyle Hidden` | fenêtre cachée |
| `-nologo` | `-NoLogo` | pas de bannière |
| `-noni` | `-NonInteractive` | pas d'invite interactive |
| `-c "..."` | `-Command` | exécute une commande |
| `-enc <b64>` | `-EncodedCommand` | commande base64 (UTF-16LE) |
| `-File x.ps1` | `-File` | exécute un script |

Exemple complet furtif :
```powershell
powershell -nop -w hidden -ep bypass -c "IEX(...)"
```

## Alternative : dans la session courante (sans relancer)
```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force
```
`-Scope Process` = seulement cette session, ne persiste pas → pas besoin d'admin.

> [!NOTE] `-ep bypass` ≠ AMSI
> Bypass règle l'**execution policy** (qui n'est PAS une sécu). Ça ne contourne PAS **AMSI** ni **AppLocker**. Si un script est bloqué malgré `-ep bypass`, c'est AMSI/AppLocker → cf [[01 - Charger PowerView]].

Voir aussi [[01 - Charger PowerView]] · [[00 - Index]]
