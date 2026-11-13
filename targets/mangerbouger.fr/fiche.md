# Fiche cible — mangerbouger.fr (cible secondaire / plan B)

- **Programme :** Santé publique France (YesWeHack)
- **Hôtes dans le périmètre :** `mangerbouger.fr`, `sso.mangerbouger.fr`
- **`www.mangerbouger.fr` :** périmètre non confirmé
- **Statut :** reconnaissance et vérifications manuelles sur `sso.mangerbouger.fr`
- **Classes étudiées :** IDOR / contrôle d'accès (CWE-284 / CWE-639), CSRF (CWE-352), authentification et récupération de compte (Keycloak)
- **Résultat :** aucune vulnérabilité confirmée

## Pourquoi cette cible était un plan B

1. Une maintenance liée à la sécurité était signalée sur Manger Bouger.
2. L'appartenance de `www.mangerbouger.fr` au périmètre n'était pas confirmée.
3. Le sous-domaine `sso.mangerbouger.fr` était explicitement autorisé et permettait d'observer le fonctionnement de l'authentification.

## Technologies observées

| Élément | Détail |
|---|---|
| Authentification | `sso.mangerbouger.fr` — Keycloak / OpenID Connect |
| Données de compte | Pages accessibles depuis `www.mangerbouger.fr` |
| Redirection | `mangerbouger.fr` → `www.mangerbouger.fr` (HTTP 301) |
| Parcours d'authentification | `/auth/realms/spf-mb-dev/login-actions/authenticate` |
| Récupération de compte | `/auth/realms/spf-mb-dev/login-actions/reset-credentials` |
| Outils | Microsoft Edge, Burp Suite Community, FoxyProxy |

## Comptes de test

Deux comptes de test ont été créés avec des alias `+ywhspf`.

Les identifiants, mots de passe, cookies et paramètres de session ne sont pas conservés dans le dépôt Git.

## Fonctionnalités étudiées

| Fonctionnalité | Hôte | Analyse | Résultat |
|---|---|---|---|
| Favoris | `www.mangerbouger.fr` | IDOR envisagée | Non testée activement, périmètre non confirmé |
| Connexion | `sso.mangerbouger.fr` | Observation des requêtes HTTP | Fonctionnement observé |
| Modification d'email | Parcours de gestion de compte Keycloak | Piste CSRF explorée | Non démontrée |
| Mot de passe oublié | `sso.mangerbouger.fr` | Analyse du formulaire et des réponses HTTP | Aucune vulnérabilité démontrée |
| Vulnérabilité Keycloak potentielle | `sso.mangerbouger.fr` | Piste CVE-2026-18963 envisagée | Applicabilité non établie, version inconnue |

## Investigations du 13/11/2026

### 1. Analyse de l'authentification

Nous avons utilisé Burp Suite pour observer les requêtes et réponses HTTP du service d'authentification Keycloak.

Les parcours de connexion et de récupération de compte ont été identifiés.

### 2. Piste CSRF

Nous avons étudié le fonctionnement de la modification d'adresse email afin de rechercher une éventuelle absence de protection CSRF.

Les observations n'ont pas permis de démontrer une modification non autorisée.

**Conclusion :** aucune vulnérabilité CSRF confirmée.

### 3. Récupération du mot de passe

Nous avons étudié le formulaire « Mot de passe oublié » et son fonctionnement.

Observations :

- Le formulaire utilise le parcours `reset-credentials` de Keycloak.
- Le serveur retourne des réponses HTTP 200.
- Une demande a affiché un message indiquant qu'un email de réinitialisation devrait être envoyé.
- Aucun email de réinitialisation n'a été reçu pendant notre vérification.
- Une tentative de connexion a retourné un message d'identifiants invalides.

Ces observations ne permettent pas de conclure à une prise de contrôle de compte ni à une vulnérabilité.

### 4. Piste CVE Keycloak

Nous avons envisagé la piste CVE-2026-18963 en lien avec le mécanisme de récupération de compte.

Cependant, la version de Keycloak n'a pas été identifiée et l'applicabilité de cette CVE n'a pas été établie.

**Conclusion :** piste clôturée sans vulnérabilité démontrée.

## Décision finale

Les vérifications manuelles réalisées sur `sso.mangerbouger.fr` n'ont révélé aucune vulnérabilité confirmée.

Les tests IDOR sur les pages de compte de `www.mangerbouger.fr` n'ont pas été poursuivis en raison de l'ambiguïté du périmètre et du contexte de maintenance.

Cette cible reste documentée comme piste secondaire du projet, sans signalement de vulnérabilité sur YesWeHack.
