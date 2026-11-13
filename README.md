# Bug Bounty — Santé publique France

- **Projet Master 2 Cybersécurité — Université de Caen Normandie**
- **Binôme (groupe 4) :** Imane EL GHAIT & Melissa GUERAOUNI
- **Programme :** [Santé Publique France — YesWeHack](https://yeswehack.com/programs/sante-publique-france-bugbounty-public-program)
- **Période :** novembre 2026

> Ce dépôt est notre **carnet de travail** : notes, analyses, journal et preuves au fil de l'eau.
> Le **rapport** est le livrable principal et se lit indépendamment de ce dépôt.

## Contexte

Démarche de bug bounty sur le programme public Santé publique France, menée en binôme sur une cible principale et une classe de vulnérabilité, dans le respect strict des règles du programme (User-Agent `YWHSPF`, pas d'outils automatisés, aucune donnée modifiée).

- **Cible principale :** `www.tabac-info-service.fr`
- **Classe principale :** XSS réfléchie (CWE-79)
- **Classes analysées et écartées :** contrôle d'accès / IDOR (CWE-284/639), fuite via messages d'erreur (CWE-209)

## Résultat

Aucune vulnérabilité retenue. Les points d'entrée testés sont correctement protégés. Conformément aux attendus, le travail documente la **démarche** et les **défenses observées**.

| Point d'entrée | Résultat |
|---|---|
| Recherche principale | caractères `< > " '` échappés |
| Autocomplétion (`/autocomplete/article/`) | entrée non reflétée, réponse JSON |
| Filtre `theme` (Questions/Réponses) | encodage URL ; `"` → erreur 500 |
| Filtre `page` | erreur 500 sur valeur non numérique |
| Recherche de tabacologue | déléguée à Google Maps (hors périmètre) |

## Organisation du dépôt

- `scope/` — périmètre (`in-scope.txt`, `out-of-scope.txt`) et règles (`regles.md`)
- `targets/www.tabac-info-service.fr/` — la cible :
  - `fiche.md` — carte d'identité et surface d'attaque
  - `journal.md` — journal de bord, une entrée par séance
  - `checklist-web.md` — checklist de test
  - `notes/` — analyses détaillées (00 à 04)
  - `evidences/` — captures d'écran des tests clés
- `docs/`, `templates/` — méthodologie et modèles fournis par l'encadrant

## Notes d'analyse

- [00 — Environnement de travail](targets/www.tabac-info-service.fr/notes/00-environnement.md)
- [01 — Choix de la cible et de la classe](targets/www.tabac-info-service.fr/notes/01-choix-de-la-classe.md)
- [02 — XSS recherche principale](targets/www.tabac-info-service.fr/notes/02-recherche-principale-xss.md)
- [03 — XSS autocomplétion](targets/www.tabac-info-service.fr/notes/03-autocompletion-xss.md)
- [04 — XSS filtres Questions/Réponses](targets/www.tabac-info-service.fr/notes/04-filtres-questions-reponses-xss.md)
