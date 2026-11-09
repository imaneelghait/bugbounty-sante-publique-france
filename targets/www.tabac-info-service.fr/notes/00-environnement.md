# 00 — Environnement de travail

**Date :** 2026-11-06 (mis à jour le 2026-11-09)

## 1. Poste de test

| Élément | Choix | Pourquoi |
|---|---|---|
| Navigateur de test | **Firefox**, proxy `127.0.0.1:8080` toujours activé | Tout le trafic de test passe par Burp |
| Navigateur personnel | **Edge / Chrome**, sans proxy | Recherches, GitHub, YesWeHack : on ne mélange pas avec les tests |
| Proxy d'interception | **Burp Suite Community** (v2026.10.1) + certificat CA installé dans Firefox | Voir, modifier et rejouer les requêtes ; historique comme preuve |
| Dépôt | Copie **privée** du modèle `Karlblock/BugBounty` (import GitHub) | Un fork d'un dépôt public est forcément public ; le programme interdit toute divulgation |
| Données sensibles | Dossier local `BugBounty_SPF/` | Identifiants de test jamais dans le dépôt |

## 2. User-Agent obligatoire (`YWHSPF`)

Règle Burp *Match and replace* :

- Type : `Request header`
- Match : `^User-Agent: (.*)$`
- Replace : `User-Agent: $1 YWHSPF`

**Difficultés rencontrées :**
- Burp affiche par défaut la requête **originale** dans l'historique ; la version modifiée se lit via *Original request → Auto-modified request*.
- En projet temporaire (Community), la règle est perdue à la fermeture : elle est sauvegardée (`Save project settings`) et rechargée à chaque démarrage (`Load from configuration file`).

**Vérification (2026-11-06) :** `User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:159.0) Gecko/20100101 Firefox/159.0 YWHSPF`

## 3. Comptes de test

- Alias `@yeswehack.ninja` introuvable dans nos comptes YesWeHack ; la fiche dit « *you can* » (possibilité, pas obligation). Nous utilisons des adresses dédiées avec le suffixe `+ywhspf`.
- Données saisies fictives.

| Site | Résultat |
|---|---|
| `www.tabac-info-service.fr` | Aucune création de compte dans le périmètre (inscription « Mois sans tabac » → sous-domaine hors périmètre ; application mobile hors périmètre) |
| `mangerbouger.fr` | Comptes via `sso.mangerbouger.fr` (dans le périmètre). Compte A : Imane ✅ — Compte B : Melissa ⏳ |

## 4. Points de vigilance

- Des liens officiels mènent à des sous-domaines hors périmètre (`mois-sans-tabac.tabac-info-service.fr`) : on vérifie l'adresse avant chaque action.
- Mangerbouger en maintenance de sécurité (bandeau du 2026-11-06) : pas de tests actifs tant qu'il est affiché.
- `www.mangerbouger.fr` vs `mangerbouger.fr` : inclusion du `www` à confirmer.
- Connexion Mangerbouger : `sso.mangerbouger.fr`, chemin `/auth/realms/spf-mb-dev/protocol/openid-connect/…` (Keycloak, OpenID Connect).
