# Cheatsheet - Quel outil quand

> [!summary] Objectif
> Choisir le BON outil selon l'objectif. Erreur n°1: confondre les outils de la couche **AD** (annuaire) et ceux de la couche **HÔTE** (la machine). Noms proches ≠ même job.

## Le principe: 2 couches, 2 familles

| Couche | Ce que tu attaques | Outils |
|---|---|---|
| **Identité / AD** | domaine, users, groupes, ACL, delegation, trusts, tickets | PowerView · AD module · BloodHound · Rubeus |
| **Hôte / machine** | services locaux, tâches, perms, LSASS, apps (Jenkins) | PowerUp · winPEAS · PrivEscCheck · Seatbelt · mimikatz/SafetyKatz |

## PowerUp vs PowerView — LA confusion à tuer
- **PowerView** = énumération **AD** (annuaire): `Get-Domain*`, ACL, SPN, delegation, trusts. **Ne touche pas** la machine locale.
- **PowerUp** = privesc **locale Windows**: services faibles, unquoted paths, autoruns, registry. **Ne touche pas** l'AD.
- Noms qui se ressemblent, **jobs opposés** (AD ↔ hôte).

## Table de décision (par objectif)

| Je veux… | Outil | Couche |
|---|---|---|
| Énumérer le domaine (users, groupes, ACL, SPN, trusts) | **PowerView** (ou AD module signé si Constrained Language Mode) | AD |
| Voir les chemins d'attaque AD (qui contrôle quoi, path→DA) | **BloodHound** + SharpHound | AD |
| Trouver une **privesc locale** (rapide, ciblé services/AD) | **PowerUp** `Invoke-AllChecks` | Hôte |
| Audit privesc local **exhaustif** (si PowerUp ne trouve rien) | **winPEAS** (verbeux) · **PrivEscCheck** | Hôte |
| Inventaire host (tokens, .NET, AV, creds sur disque) | **Seatbelt** | Hôte |
| **Où suis-je déjà admin** (PSRemoting/WMI/sessions) | **Find-PSRemotingLocalAdminAccess** · **Find-WMILocalAdminAccess** · **Invoke-SessionHunter** | Hôte↔AD |
| **Dumper** creds/hash/tickets (besoin admin/SYSTEM) | **SafetyKatz** / **mimikatz** / Invoke-Mimi | Hôte |
| Forger / rejouer des tickets Kerberos | **Rubeus** | AD |
| Surface locale brute (ports, services, apps → Jenkins, SQL) | `Get-CimInstance win32_service` · `Get-NetTCPConnection` · `netstat` | Hôte |

## Règle mentale (3 questions)
1. **Monter en privilège SUR une machine ?** → outils **HÔTE**. PowerUp d'abord (ciblé/rapide), winPEAS si rien.
2. **Où aller / qui abuser dans le domaine ?** → outils **AD**. PowerView pour requêter précis, BloodHound pour voir les chemins.
3. **J'ai un accès admin, je veux des secrets ?** → **SafetyKatz/mimikatz** (dump LSASS).

## Ordre-type sur une NOUVELLE machine
```text
1. Contexte      : whoami /all ; whoami /priv
2. Surface locale: services non-MS, ports en écoute, apps
3. Privesc locale: PowerUp Invoke-AllChecks  (→ winPEAS si rien)
4. Si admin/SYSTEM: dump creds (SafetyKatz)  → hash/creds de DOMAINE
5. Énum AD       : PowerView + BloodHound (avec le meilleur compte en main)
6. Choisir le saut suivant (lateral / abuse ACL / delegation…)
```

> [!warning] Compte LOCAL ≠ DOMAINE — et « pourquoi j'ai john »
> `Invoke-ServiceAbuse` **exécute une commande en tant que compte du service (SYSTEM)**. Sans `-Command`, PowerUp lance sa charge par défaut = créer l'admin **local** `john`. Donc john = **preuve** que tu exécutes en SYSTEM, **pas** l'objectif.
> Un admin **local** = contrôle de la **machine** seulement (jamais dans BloodHound, zéro portée AD). Sa **seule** valeur : dump LSASS → voler des **creds de DOMAINE**, qui eux font avancer.
> **Le plus direct :** saute john, mets ta charge d'emblée → `Invoke-ServiceAbuse -Name <svc> -Command "<dump / reverse shell>"` tourne en SYSTEM. (`runas .\john` échoue en shell non-interactif ; `RunasCs`/`PsExec` absents de `C:\AD\Tools`.)

## Voir aussi
[[Cheatsheet - Privilege escalation]] · [[Cheatsheet - Reconnaissance AD]] · [[Cheatsheet - BloodHound SharpHound]] · [[Cheatsheet - Post-exploitation]] · [[Cheatsheet - Chemins des outils]]
