# Analyse de l'authentification Keycloak — Manger Bouger

**Date :** 13/11/2026  
**Étudiante :** Melissa GUERAOUNI  
**Programme :** Santé publique France — YesWeHack  
**Cible :** `sso.mangerbouger.fr`  
**Outils :** Microsoft Edge, Burp Suite Community, FoxyProxy  
**Statut :** Tests terminés — aucune vulnérabilité confirmée

## 1. Objectif

Dans le cadre de notre projet de Bug Bounty, nous avons étudié le système d'authentification du site Manger Bouger.

Le site utilise Keycloak pour gérer les connexions et la récupération des comptes.

Notre objectif était d'observer les mécanismes de sécurité et d'identifier d'éventuelles faiblesses, notamment concernant :

- La modification d'adresse email (CSRF).
- La réinitialisation du mot de passe.
- Une éventuelle vulnérabilité connue de Keycloak.

## 2. Environnement de test

Les vérifications ont été réalisées manuellement avec Microsoft Edge et Burp Suite Community.

FoxyProxy permettait de faire passer le trafic du navigateur par Burp Suite afin d'observer les échanges HTTP.

Nous avons utilisé un compte de test créé pour le projet.

Les paramètres de session, cookies, mots de passe et adresses email ne sont pas conservés dans ce dépôt.

Les investigations ont été limitées aux fonctionnalités étudiées, sans scanner automatisé ni tentative d'accès à des comptes tiers.

## 3. Reconnaissance

Nous avons commencé par explorer le parcours de connexion de Manger Bouger.

L'authentification est assurée par le sous-domaine :

`sso.mangerbouger.fr`

Le service utilise notamment les points d'entrée suivants :

| Point d'entrée | Fonction |
|---|---|
| `/auth/realms/spf-mb-dev/login-actions/authenticate` | Étapes du parcours d'authentification |
| `/auth/realms/spf-mb-dev/login-actions/reset-credentials` | Récupération du mot de passe |

Ces chemins ont été identifiés dans les requêtes et les réponses HTTP observées avec Burp Suite.

## 4. Analyse de la protection CSRF

### Objectif

Nous avons envisagé une vulnérabilité CSRF sur la fonctionnalité de modification d'adresse email.

Une attaque CSRF consiste à provoquer une action non souhaitée dans la session authentifiée d'un utilisateur.

### Méthodologie

Nous avons étudié le fonctionnement de la modification d'email dans le parcours de gestion du compte et observé les échanges HTTP associés.

Nous cherchions à comprendre comment Keycloak associait les actions à une session utilisateur.

### Résultat

Les vérifications effectuées n'ont pas permis de démontrer qu'une modification d'adresse email pouvait être déclenchée sans autorisation.

**Conclusion : aucune vulnérabilité CSRF confirmée.**

## 5. Analyse de la récupération du mot de passe

### Objectif

Nous avons ensuite étudié le mécanisme « Mot de passe oublié » pour comprendre le traitement des demandes de réinitialisation.

### Méthodologie

1. Accéder au formulaire « Mot de passe oublié ».
2. Observer les requêtes HTTP avec Burp Suite.
3. Effectuer une demande sur notre compte de test.
4. Examiner les réponses HTTP et les messages affichés.

### Observations

**Observation 1 — Formulaire de réinitialisation**

Le serveur retourne une réponse `HTTP/2 200 OK` contenant le formulaire « Mot de passe oublié ».

Le formulaire utilise le parcours Keycloak `reset-credentials` avec une méthode POST.

**Observation 2 — Confirmation de demande**

Après une demande, une page de connexion a affiché le message :

> Vous devriez recevoir rapidement un courriel avec de plus amples instructions.

La réponse HTTP était `200 OK`.

Ce message confirme que l'application a présenté une confirmation à l'utilisateur, mais ne prouve pas que l'email a effectivement été envoyé.

**Observation 3 — Réception de l'email**

Nous n'avons pas reçu d'email de réinitialisation pendant notre vérification.

Nous n'avons pas pu déterminer si cela provenait d'un problème d'envoi, de réception ou de configuration du compte.

**Observation 4 — Connexion**

Une réponse HTTP contenait le message :

> Nom d'utilisateur ou mot de passe invalide.

Cette réponse correspond à un échec d'authentification.

### Résultat

Aucune réinitialisation de mot de passe non autorisée ni prise de contrôle de compte n'a été démontrée.

## 6. Étude d'une piste CVE Keycloak

Nous avons également envisagé une piste liée à la référence `CVE-2026-18963`, supposée concerner le mécanisme de récupération de compte Keycloak.

Cette référence a été utilisée comme hypothèse de recherche et non comme vulnérabilité confirmée.

Nous n'avons pas pu identifier la version exacte de Keycloak utilisée par Manger Bouger, ni établir que l'instance était concernée.

Aucune preuve d'exploitation n'a été obtenue.

**Conclusion : piste non confirmée et clôturée.**

## 7. Synthèse des résultats

| Piste | Résultat | Statut |
|---|---|---|
| CSRF — modification d'email | Modification non autorisée non démontrée | Non confirmée |
| Récupération du mot de passe | Fonctionnement HTTP observé | Aucune vulnérabilité démontrée |
| CVE-2026-18963 | Version et applicabilité non établies | Non confirmée |
| Prise de contrôle de compte | Aucun scénario démontré | Non confirmée |

## 8. Conclusion

Cette investigation nous a permis de mieux comprendre le fonctionnement de Keycloak et d'apprendre à analyser les requêtes HTTP liées à l'authentification.

L'utilisation de Burp Suite a été utile pour observer les formulaires, les paramètres de session et les réponses du serveur.

Même si nous n'avons pas découvert de vulnérabilité, cette recherche nous a permis de suivre une méthodologie, d'étudier plusieurs hypothèses et de distinguer une observation technique d'une véritable preuve de vulnérabilité.

Aucun signalement n'a été effectué sur YesWeHack pour ces pistes.
