# 00 — Observation Mangerbouger (plan B, non testé)

- **Date :** 2026-11-13
- **Statut :** reconnaissance uniquement, aucun test actif
- **Classe envisagée :** IDOR / contrôle d'accès (CWE-284 / CWE-639)

## Observations (usage normal, sans injection)

- **Authentification** sur `sso.mangerbouger.fr` (Keycloak / OpenID Connect).
- **Données de compte** servies depuis `www.mangerbouger.fr` :
  `GET /mon-compte/mes-favoris` sur `Host: www.mangerbouger.fr`.
- **Pas d'identifiant manipulable** : l'URL `/mon-compte/mes-favoris` ne contient aucun identifiant d'objet. L'utilisateur est identifié par un **cookie de session**, pas par un paramètre d'URL → pas de vecteur IDOR direct sur cette page.

## Question de périmètre : `www.mangerbouger.fr`

- La fiche du programme liste `mangerbouger.fr` (sans `www`) et `sso.mangerbouger.fr` ; pas de wildcard.
- `www.mangerbouger.fr` n'est pas listé explicitement.
- **Preuve de la redirection** (apex → www) :
  ```
  GET / HTTP/1.1
  Host: mangerbouger.fr

  HTTP/2 301 Moved Permanently
  Location: https://www.mangerbouger.fr/
  ```
- Le périmètre est défini par le **programme** (Santé publique France / YesWeHack), nous appliquons la **lecture prudente** : `www.mangerbouger.fr` est traité comme non confirmé, donc non testé.

## Décision

Tant que la maintenance de sécurité n'est pas terminée et le périmètre `www.` n'est pas confirmé par le programme, **aucun test actif** n'est mené sur Mangerbouger.
