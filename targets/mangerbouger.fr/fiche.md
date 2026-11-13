# Fiche cible — mangerbouger.fr (cible secondaire / plan B)

- **Programme :** Santé publique France (YesWeHack)
- **Hôtes dans le périmètre :** `mangerbouger.fr`, `sso.mangerbouger.fr`
- **`www.mangerbouger.fr` :** non confirmé 
- **Statut :** reconnaissance uniquement, **aucun test actif**
- **Classe envisagée :** IDOR / contrôle d'accès (CWE-284 / CWE-639)

## Pourquoi plan B (non activé)

1. Maintenance de sécurité en cours (bandeau présent depuis le 06/11).
2. Périmètre de `www.mangerbouger.fr` non confirmé.

## Technologies observées

| Élément | Détail |
|---|---|
| Authentification | `sso.mangerbouger.fr` — Keycloak / OpenID Connect |
| Données de compte | servies depuis `www.mangerbouger.fr` |
| Redirection | `mangerbouger.fr` → `www.mangerbouger.fr` (HTTP 301) |

## Comptes de test

Deux comptes prévus (un par membre), créés avec des alias `+ywhspf`.
Identifiants et cookies de session **non versionnés** (conformément aux règles du dépôt).

## Surface envisagée (non testée)

| Fonctionnalité | Hôte | Classe | Testé |
|---|---|---|---|
| Favoris (`/mon-compte/mes-favoris`) | `www.mangerbouger.fr` | IDOR | Non — pas d'identifiant manipulable (session), hôte non confirmé |
| Connexion | `sso.mangerbouger.fr` | Auth / OAuth | Non — hors de la classe principale |

## Décision

Aucun test actif. Cible conservée en plan B, documentée pour la cohérence de la démarche.
