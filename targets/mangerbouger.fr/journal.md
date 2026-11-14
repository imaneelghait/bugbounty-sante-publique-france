# Journal de bord — Manger Bouger

**Programme :** Santé publique France — YesWeHack  
**Étudiante :** Melissa GUERAOUNI  
**Cible :** `sso.mangerbouger.fr`  
**Période :** 13–14 novembre 2026  
**Outils :** Microsoft Edge, FoxyProxy, Burp Suite Community

## 13/11/2026 — Reconnaissance et analyse de Keycloak

### Objectif

Étudier le fonctionnement de l'authentification Manger Bouger et rechercher d'éventuelles faiblesses dans la gestion des comptes.

### Travail réalisé

- Identification du service d'authentification Keycloak sur `sso.mangerbouger.fr`.
- Observation des requêtes HTTP avec Burp Suite.
- Analyse du formulaire « Mot de passe oublié ».
- Envoi d'une demande de réinitialisation sur notre compte de test.
- Étude d'une éventuelle piste CSRF (CWE-352).
- Recherche exploratoire concernant une éventuelle vulnérabilité connue de Keycloak.

### Observations

- Le parcours de récupération du mot de passe utilise `/login-actions/reset-credentials`.
- Le serveur a retourné des réponses HTTP 200.
- Après la demande de réinitialisation, le message suivant est apparu : « Vous devriez recevoir rapidement un courriel avec de plus amples instructions. »
- Aucun email de réinitialisation n'a été reçu pendant notre vérification.
- Une tentative de connexion a affiché un message d'identifiants invalides.
- La version exacte de Keycloak n'a pas été identifiée.

### Résultat

Aucune réinitialisation non autorisée ni vulnérabilité CSRF n'a été démontrée.

La piste liée à une éventuelle CVE Keycloak n'a pas pu être confirmée.

## 14/11/2026 — Analyse des sessions Keycloak

### Objectif

Comprendre le rôle des paramètres de session dans le parcours d'authentification et approfondir la piste CSRF.

### Travail réalisé

1. Observation du parcours de connexion dans l'historique HTTP de Burp Suite.
2. Identification d'une requête GET vers `/protocol/openid-connect/auth`, avec une réponse HTTP 200.
3. Observation d'une requête POST vers `/login-actions/authenticate`, avec une réponse HTTP 302.
4. Identification des paramètres `state`, `session_code`, `execution`, `client_id` et `tab_id`.
5. Observation des cookies `AUTH_SESSION_ID`, `AUTH_SESSION_ID_LEGACY` et `KC_RESTART`.
6. Reprise d'une requête POST dans Burp Repeater.
7. Remplacement de `session_code` par `invalid-session-test` et des identifiants par des valeurs fictives.
8. Envoi d'une seule requête modifiée.
9. Comparaison du comportement de l'URL de redirection dans une session Edge existante et dans une fenêtre InPrivate.

### Observations

**Parcours de connexion normal :**

Une redirection vers `/login_check` a été observée avec les paramètres `state`, `session_state` et `code`.

**Requête modifiée dans Burp Repeater :**

- Réponse HTTP/2 302.
- Redirection vers `/login-actions/authenticate`.
- Nouveau cookie `AUTH_SESSION_ID`.
- Aucun code d'autorisation observé dans cette réponse.

**Comparaison des sessions :**

- Dans une session Edge déjà connectée : « Vous êtes déjà connecté ».
- Dans une fenêtre Edge InPrivate sans les cookies précédents : `cookieNotFoundMessage`.

### Interprétation

Le contexte de session influence le fonctionnement du parcours d'authentification Keycloak.

Cependant, plusieurs éléments de la requête ont été modifiés simultanément. Nous ne pouvons donc pas attribuer la différence de comportement uniquement à `session_code`.

Ces observations ne prouvent pas l'existence d'une vulnérabilité CSRF.

### Limites

- Aucun scénario CSRF complet n'a été exécuté.
- Aucune action non autorisée n'a été démontrée.
- Le comportement du seul paramètre `session_code` n'a pas été isolé.
- Aucun test actif n'a été réalisé sur `www.mangerbouger.fr`, dont le statut dans le périmètre restait à confirmer.

### Résultat

Aucune vulnérabilité CSRF (CWE-352) confirmée.

Les résultats concernent uniquement les requêtes et les scénarios effectivement étudiés.

## Bilan de l'investigation

Cette investigation nous a permis de mieux comprendre le fonctionnement de Keycloak, le parcours OpenID Connect et la gestion des sessions.

Les tests ont été réalisés manuellement avec Burp Suite, sans scanner automatisé.

Aucune vulnérabilité exploitable n'a été confirmée et aucun signalement n'a été soumis sur YesWeHack.

Les analyses détaillées et les captures sont conservées dans les dossiers `notes/` et `evidences/` de cette cible.
