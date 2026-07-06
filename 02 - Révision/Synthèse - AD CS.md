# Synthèse - AD CS

> [!summary] Résumé
> Abus de certificats. Cours lié : [[Attaques AD]]. Fiche : [[Cheatsheet - AD CS et certificats]].

## Mécanique

Une **CA** délivre des certificats selon des **templates**. Deux leviers décident de l'impact : qui peut **enroll** + ce que le template autorise.

## ESC à connaître

- **ESC1** : template avec **`ENROLLEE_SUPPLIES_SUBJECT`** (SAN arbitraire) + EKU d'authentification client + enrôlement ouvert aux Domain Users → demander un certificat **au nom d'un DA** → TGT DA.
- **ESC3** : template *Enrollment Agent* → obtenir un certificat d'agent puis demander un certificat **au nom d'un autre** compte → même effet.

## À noter précisément

Nom du template, EKU, droits d'enrôlement, flag `ENROLLEE_SUPPLIES_SUBJECT`. Le certificat → TGT via PKINIT → DA/EA. Court-circuite la chaîne classique : un simple Domain User suffit.

## Notes associées

- [[Cheatsheet - AD CS et certificats]]
- [[Attaques AD]]
- [[2. Attack flow]]
