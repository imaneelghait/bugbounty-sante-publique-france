
# Analyse de l'authentification Keycloak — Manger Bouger

**Dates :** 13 et 14/11/2026  
**Étudiante :** Melissa GUERAOUNI  
**Programme :** Santé publique France — YesWeHack  
**Cible :** `sso.mangerbouger.fr`  
**Outils :** Microsoft Edge, Burp Suite Community, FoxyProxy  
**Statut :** Tests terminés — aucune vulnérabilité confirmée

## 1. Objectif

Dans le cadre de notre projet de Bug Bounty, nous avons étudié le système d'authentification du site Manger Bouger.

Le site utilise Keycloak pour gérer les connexions et la récupération des comptes.

Notre objectif était d'observer les mécanismes de sécurité et d'étudier plusieurs pistes :

- La gestion des sessions et une éventuelle vulnérabilité CSRF (CWE-352).
- La récupération du mot de passe.
- Une éventuelle vulnérabilité connue de Keycloak.

Les tests ont été limités au périmètre autorisé par le programme YesWeHack.

## 2. Environnement de test

Les vérifications ont été réalisées manuellement avec Microsoft Edge et Burp Suite Community.

FoxyProxy permettait de faire passer le trafic du navigateur par Burp Suite afin d'observer les échanges HTTP.

Nous avons utilisé un compte de test créé pour le projet.

Les paramètres sensibles de session, cookies, mots de passe et adresses email ne sont pas reproduits dans ce document.

Les investigations ont été réalisées sans scanner automatisé ni tentative d'accès à des comptes tiers.

## 3. Reconnaissance

Nous avons commencé par explorer le parcours de connexion de Manger Bouger.

L'authentification est assurée par le sous-domaine :

`sso.mangerbouger.fr`

Le service utilise notamment les points d'entrée suivants :

| Point d'entrée | Fonction |
|---|---|
| `/auth/realms/spf-mb-dev/protocol/openid-connect/auth` | Initialisation de l'authentification |
| `/auth/realms/spf-mb-dev/login-actions/authenticate` | Traitement de l'authentification |
| `/auth/realms/spf-mb-dev/login-actions/reset-credentials` | Récupération du mot de passe |

Ces chemins ont été identifiés dans les échanges HTTP observés avec Burp Suite.

Nous avons également identifié plusieurs paramètres utilisés pendant l'authentification :

- `state`
- `session_code`
- `execution`
- `client_id`
- `tab_id`

Les cookies observés comprenaient notamment `AUTH_SESSION_ID`, `AUTH_SESSION_ID_LEGACY` et `KC_RESTART`.

## 4. Analyse de la piste CSRF

### 4.1 Objectif

Nous avons envisagé une vulnérabilité CSRF (CWE-352) dans le système d'authentification et de gestion des sessions.

Une attaque CSRF consiste à provoquer une action non souhaitée dans la session authentifiée d'un utilisateur.

Nous souhaitions comprendre comment Keycloak associait les requêtes d'authentification à une session utilisateur.

### 4.2 Méthodologie

Nous avons observé les requêtes HTTP liées à la connexion et identifié les paramètres utilisés pour suivre le parcours d'authentification.

Nous avons ensuite étudié le comportement du serveur lorsqu'un paramètre de session était modifié.

Aucun scénario complet d'attaque CSRF n'a été exécuté.

### 4.3 Résultat

Les observations réalisées n'ont pas permis de démontrer qu'une action pouvait être déclenchée à l'insu d'un utilisateur authentifié.

**Conclusion : aucune vulnérabilité CSRF confirmée.**

## 5. Analyse de la récupération du mot de passe

### 5.1 Objectif

Nous avons également étudié le mécanisme « Mot de passe oublié » pour comprendre le traitement des demandes de réinitialisation.

### 5.2 Méthodologie

1. Accéder au formulaire « Mot de passe oublié ».
2. Observer les requêtes HTTP avec Burp Suite.
3. Effectuer une demande sur notre compte de test.
4. Examiner les réponses HTTP et les messages affichés.

### 5.3 Observations

**Observation 1 — Formulaire de réinitialisation**

Le serveur retourne une réponse `HTTP/2 200 OK` contenant le formulaire « Mot de passe oublié ».

Le formulaire utilise le parcours Keycloak `reset-credentials` avec une méthode POST.

**Observation 2 — Confirmation de demande**

Après une demande, une page de connexion a affiché le message :

> Vous devriez recevoir rapidement un courriel avec de plus amples instructions.

La réponse HTTP était `200 OK`.

Ce message indique que l'application a présenté une confirmation à l'utilisateur, mais ne prouve pas que l'email a effectivement été envoyé.

**Observation 3 — Réception de l'email**

Nous n'avons pas reçu d'email de réinitialisation pendant notre vérification.

Nous n'avons pas pu déterminer si cela provenait d'un problème d'envoi, de réception ou de configuration du compte.

**Observation 4 — Connexion**

Une réponse HTTP contenait le message :

> Nom d'utilisateur ou mot de passe invalide.

Cette réponse correspond à un échec d'authentification.

### 5.4 Résultat

Aucune réinitialisation de mot de passe non autorisée ni prise de contrôle de compte n'a été démontrée.

## 6. Étude d'une piste CVE Keycloak

Nous avons également envisagé une piste liée à la référence `CVE-2026-18963`, supposée concerner le mécanisme de récupération de compte Keycloak.

Cette référence a été utilisée comme hypothèse de recherche et non comme vulnérabilité confirmée.

Nous n'avons pas pu identifier la version exacte de Keycloak utilisée par Manger Bouger, ni établir que l'instance était concernée.

Aucune preuve d'exploitation n'a été obtenue.

**Conclusion : piste non confirmée et clôturée.**

## 7. Analyse des sessions Keycloak — 14/11/2026

### 7.1 Objectif

Nous avons poursuivi l'analyse de l'authentification Keycloak sur `sso.mangerbouger.fr`, en étudiant les paramètres de session et une éventuelle piste CSRF (CWE-352).

Les tests ont été réalisés manuellement avec Microsoft Edge, FoxyProxy et Burp Suite Community.

### 7.2 Observation du parcours d'authentification

Dans l'historique HTTP de Burp Suite, nous avons observé :

| Requête | Réponse |
|---|---|
| `GET /auth/realms/spf-mb-dev/protocol/openid-connect/auth` | HTTP 200 |
| `POST /auth/realms/spf-mb-dev/login-actions/authenticate` | HTTP 302 |

Lors d'une connexion normale, Keycloak redirige vers :

`https://www.mangerbouger.fr/login_check`

Cette redirection contient les paramètres :

- `state`
- `session_state`
- `code`

Elle correspond au parcours d'authentification OpenID Connect avec un code d'autorisation.

Cette redirection a uniquement été observée dans Burp Suite, sans test actif sur `www.mangerbouger.fr`.

### 7.3 Modification du paramètre session_code

Nous avons repris une requête POST d'authentification dans Burp Repeater.

Le paramètre a été remplacé par :

`session_code=invalid-session-test`

Les identifiants ont également été remplacés par des valeurs fictives.

Une seule requête modifiée a été envoyée.

**Résultat observé :**

- Réponse `HTTP/2 302`.
- Redirection vers `/login-actions/authenticate`.
- Nouveau cookie `AUTH_SESSION_ID`.
- Aucun code d'autorisation observé dans cette réponse.

Contrairement à la connexion normale, nous n'avons pas observé de redirection vers `/login_check` avec un code d'autorisation.

Cependant, plusieurs éléments de la requête avaient été modifiés.

Nous ne pouvons donc pas attribuer cette différence de comportement au seul paramètre `session_code`.

### 7.4 Comparaison des sessions

Nous avons ensuite ouvert l'URL de redirection dans deux contextes différents.

**Premier cas — Session Edge déjà connectée**

Le navigateur affichait :

> Vous êtes déjà connecté.

**Deuxième cas — Fenêtre Edge InPrivate**

Sans les cookies de la session précédente, le navigateur affichait :

`cookieNotFoundMessage`

Ces observations montrent que le contexte de session influence le parcours d'authentification.

Elles ne constituent cependant pas une preuve de vulnérabilité CSRF.

### 7.5 Limites des tests

- Aucun scénario CSRF complet n'a été exécuté.
- Aucune action non autorisée n'a été démontrée.
- Le comportement du paramètre `session_code` n'a pas été isolé.
- Aucun test actif n'a été réalisé sur `www.mangerbouger.fr`, dont le statut dans le périmètre restait à confirmer.

### 7.6 Conclusion

Cette investigation nous a permis de mieux comprendre le fonctionnement d'OpenID Connect et la gestion des sessions dans Keycloak.

Aucune vulnérabilité CSRF (CWE-352) n'a été confirmée.

Les résultats restent limités aux requêtes et aux scénarios effectivement étudiés.

## 8. Synthèse des résultats

| Piste étudiée | Observation | Statut |
|---|---|---|
| Authentification OpenID Connect | GET 200 et POST 302 observés | Parcours documenté |
| Modification de `session_code` | Redirection HTTP 302 et nouveau cookie | Effet isolé non démontré |
| Comparaison des sessions | Messages différents selon le contexte | Comportement observé |
| CSRF (CWE-352) | Aucune action non autorisée démontrée | Non confirmée |
| Récupération du mot de passe | Fonctionnement HTTP observé | Aucune vulnérabilité démontrée |
| CVE-2026-18963 | Version et applicabilité non établies | Non confirmée |
| Prise de contrôle de compte | Aucun scénario démontré | Non confirmée |

## 9. Conclusion générale

Cette investigation nous a permis de mieux comprendre le fonctionnement de Keycloak et d'analyser les requêtes HTTP liées à l'authentification.

L'utilisation de Burp Suite nous a permis d'observer les formulaires, les paramètres de session, les cookies et les réponses du serveur.

Nous avons notamment constaté que le comportement du parcours d'authentification dépend du contexte de session.

Cependant, une redirection HTTP ou un message d'erreur ne constitue pas à lui seul une vulnérabilité.

Il est nécessaire de pouvoir reproduire le comportement et de démontrer un impact réel.

**Aucune vulnérabilité exploitable n'a été confirmée dans le cadre de cette investigation.**

Aucun signalement n'a été effectué sur YesWeHack pour ces pistes.
