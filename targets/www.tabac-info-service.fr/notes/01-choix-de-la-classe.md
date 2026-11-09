# 01 — Choix de la cible et de la classe de vulnérabilité

**Date :** 2026-11-06
**Décision du binôme :** XSS réfléchie sur `www.tabac-info-service.fr`

## Options étudiées

| Option | Cible | Décision |
|---|---|---|
| **XSS réfléchie** | Fonctionnalités de recherche de `www.tabac-info-service.fr` | ✅ Retenue |
| IDOR / contrôle d'accès | Espace compte de `mangerbouger.fr` | Plan B |
| Authentification / OAuth | `sso.mangerbouger.fr` (`redirect_uri`) | Écartée |

## Pourquoi la XSS sur `www.tabac-info-service.fr`

- Hôte listé tel quel dans le périmètre.
- Site disponible (pas de maintenance de sécurité).
- Pas besoin de compte : les recherches sont publiques.
- Une recherche recopie le texte saisi dans la page : c'est le mécanisme d'une XSS.
- La recherche passe dans l'URL (`GET`) : une XSS pourrait toucher une autre personne via un lien, donc ce ne serait pas une Self-XSS (refusée par le programme).
- Un premier test existe déjà (`01-recherche-principale-xss.md`).

## Pourquoi pas les autres (pour l'instant)

- **IDOR sur Mangerbouger :** maintenance « liée à une faille de sécurité » en cours, et inclusion de `www.mangerbouger.fr` à confirmer.
- **OAuth sur le SSO :** même raison, et niveau technique plus avancé que notre temps disponible.

## Règles que nous nous fixons

- Tests manuels uniquement (Firefox + Burp), User-Agent `YWHSPF`.
- XSS réfléchie uniquement : rien n'est enregistré sur le site.
- Détection avec des caractères spéciaux et du texte repérable ; preuve éventuelle avec `console.log()`, jamais `alert()`.
- Seules les requêtes vers `www.tabac-info-service.fr` sont testées.
