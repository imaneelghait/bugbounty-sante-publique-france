
# Fiche cible — Manger Bouger

**Programme :** Santé publique France — YesWeHack  
**Cible secondaire :** Manger Bouger  
**Étudiantes :** Imane EL GHAIT et Melissa GUERAOUNI  
**Période :** Novembre 2026  
**Statut :** Investigation terminée — aucune vulnérabilité confirmée

## 1. Périmètre

Les hôtes suivants figurent dans le périmètre du programme :

- `mangerbouger.fr`
- `sso.mangerbouger.fr`

Le statut de `www.mangerbouger.fr` n'était pas explicitement confirmé.

Nous avons donc limité les tests actifs d'authentification à `sso.mangerbouger.fr`.

## 2. Pourquoi cette cible ?

Manger Bouger a été étudié comme cible secondaire après les recherches XSS sur Tabac Info Service.

Nous souhaitions explorer deux pistes complémentaires :

- IDOR (CWE-639) : contrôle d'accès aux ressources d'un utilisateur.
- CSRF (CWE-352) : gestion des sessions et des actions authentifiées.

Plusieurs fonctionnalités du site étaient en maintenance, ce qui a limité les possibilités de test.

## 3. Environnement

**Outils utilisés :**

- Microsoft Edge
- FoxyProxy
- Burp Suite Community
- Burp Repeater

Deux comptes de test avaient été créés avec des alias `+ywhspf`.

Les mots de passe, cookies et valeurs de session sensibles ne sont pas reproduits dans cette fiche.

## 4. Technologies et points d'entrée

| Élément | Observation |
|---|---|
| Authentification | Keycloak |
| Protocole | OpenID Connect |
| Hôte SSO | `sso.mangerbouger.fr` |
| Authentification | `/auth/realms/spf-mb-dev/login-actions/authenticate` |
| Autorisation OIDC | `/auth/realms/spf-mb-dev/protocol/openid-connect/auth` |
| Mot de passe oublié | `/auth/realms/spf-mb-dev/login-actions/reset-credentials` |
| Retour applicatif observé | `https://www.mangerbouger.fr/login_check` |

La redirection vers `www.mangerbouger.fr` a été observée dans les échanges HTTP, sans test actif sur cet hôte.

## 5. Investigation IDOR — CWE-639

Nous avons envisagé une vulnérabilité IDOR sur les fonctionnalités liées au compte utilisateur, notamment les favoris.

Cependant :

- Certaines fonctionnalités étaient en maintenance.
- Le périmètre de `www.mangerbouger.fr` restait ambigu.
- Aucun identifiant d'objet directement manipulable n'a été identifié dans l'URL de la page observée.

Nous n'avons pas poursuivi les tests actifs sur ces fonctionnalités.

**Résultat : aucune IDOR démontrée. Investigation non concluante.**

## 6. Investigation Keycloak — 13/11/2026

### Authentification

Nous avons observé les requêtes HTTP du parcours de connexion avec Burp Suite.

Le service utilise Keycloak pour gérer l'authentification.

### Récupération du mot de passe

Nous avons étudié le formulaire « Mot de passe oublié ».

Observations :

- Formulaire de réinitialisation accessible.
- Réponses HTTP 200.
- Message indiquant qu'un courriel d'instructions devrait être reçu.
- Aucun courriel reçu pendant notre vérification.
- Message d'identifiants invalides observé lors d'une tentative de connexion.

Aucune réinitialisation non autorisée ni prise de contrôle de compte n'a été démontrée.

### Piste CSRF

Nous avons envisagé une éventuelle vulnérabilité CSRF dans la gestion des comptes et des sessions.

Aucun scénario complet permettant de provoquer une action non autorisée n'a été exécuté.

**Résultat : aucune vulnérabilité CSRF confirmée.**

### Piste CVE Keycloak

Une hypothèse de recherche concernant `CVE-2026-18963` a été envisagée.

La version exacte de Keycloak n'a pas été identifiée et l'applicabilité de cette référence n'a pas été établie.

Cette piste a été clôturée sans vulnérabilité démontrée.

## 7. Analyse des sessions — 14/11/2026

### Parcours normal

Dans l'historique Burp, nous avons observé :

| Requête | Réponse |
|---|---|
| GET `/protocol/openid-connect/auth` | HTTP 200 |
| POST `/login-actions/authenticate` | HTTP 302 |

Lors d'une connexion normale, une redirection vers `/login_check` contenait les paramètres :

- `state`
- `session_state`
- `code`

Nous avons également observé les paramètres `session_code`, `execution`, `client_id` et `tab_id`.

Les cookies identifiés comprenaient `AUTH_SESSION_ID`, `AUTH_SESSION_ID_LEGACY` et `KC_RESTART`.

### Modification de session_code

Dans Burp Repeater, nous avons repris une requête POST d'authentification.

Nous avons remplacé le paramètre par :

`session_code=invalid-session-test`

Les identifiants ont également été remplacés par des valeurs fictives.

Une seule requête modifiée a été envoyée.

**Réponse observée :**

- HTTP/2 302.
- Redirection vers `/login-actions/authenticate`.
- Nouveau cookie `AUTH_SESSION_ID`.
- Aucun code d'autorisation observé dans cette réponse.

Plusieurs éléments ayant été modifiés, nous ne pouvons pas attribuer le résultat au seul paramètre `session_code`.

### Comparaison des sessions

Nous avons ouvert l'URL de redirection dans deux contextes :

**Session Edge déjà connectée :**

`Vous êtes déjà connecté`

**Fenêtre Edge InPrivate :**

`cookieNotFoundMessage`

Ces résultats montrent que le contexte de session influence le parcours d'authentification.

Ils ne constituent pas une preuve de vulnérabilité CSRF.

## 8. Synthèse

| Investigation | CWE | Résultat |
|---|---|---|
| IDOR | CWE-639 | Non démontrée, tests limités |
| CSRF | CWE-352 | Non démontrée |
| Authentification Keycloak | — | Parcours documenté |
| Récupération de mot de passe | — | Aucune vulnérabilité démontrée |
| Piste CVE Keycloak | — | Applicabilité non établie |

## 9. Limites

- Aucun scénario CSRF complet n'a été exécuté.
- Aucun accès non autorisé à des ressources d'un autre utilisateur n'a été démontré.
- L'effet isolé de `session_code` n'a pas été établi.
- La maintenance a limité certaines investigations.
- Aucun test actif n'a été réalisé sur `www.mangerbouger.fr`.

## 10. Documentation

Les détails des investigations sont disponibles dans :

- [Journal de bord](journal.md)
- [Analyse Keycloak](notes/01-analyse-keycloak.md)
- [Observations initiales](notes/00-observation-plan-b.md)
- [Captures et preuves](evidences/)

## 11. Conclusion

L'étude de Manger Bouger nous a permis de découvrir le fonctionnement de Keycloak, d'OpenID Connect et de la gestion des sessions.

Nous avons analysé plusieurs requêtes HTTP, formulé des hypothèses et documenté les résultats.

**Aucune vulnérabilité exploitable n'a été confirmée dans les scénarios étudiés.**

Aucun signalement n'a été soumis sur YesWeHack pour cette cible.
