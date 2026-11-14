
# Bilan des investigations — Findings

**Programme :** Santé publique France — YesWeHack  
**Projet :** Master 2 Cybersécurité  
**Étudiantes :** Imane EL GHAIT et Melissa GUERAOUNI  
**Période :** Novembre 2026

## Résultat général

Aucune vulnérabilité exploitable n'a été confirmée
pendant notre projet.

Nous avons étudié trois classes de vulnérabilités
en respectant les règles du programme.

## Synthèse des investigations

| Vulnérabilité recherchée | CWE | Cible | Résultat |
|---|---|---|---|
| XSS réfléchie | CWE-79 | www.tabac-info-service.fr | Non démontrée |
| IDOR | CWE-639 | Manger Bouger | Non démontrée, tests limités |
| CSRF | CWE-352 | sso.mangerbouger.fr | Non démontrée |

## 1. XSS réfléchie — CWE-79

Plusieurs points d'entrée ont été étudiés sur
`www.tabac-info-service.fr`.

Les tests ont notamment porté sur les paramètres
de recherche et certains filtres.

Nous avons observé des mécanismes d'échappement
des caractères spéciaux ainsi que certaines
réponses HTTP 500.

Aucune exécution de JavaScript injecté
n'a été démontrée.

**Statut : aucune XSS confirmée.**

## 2. IDOR — CWE-639

Nous avons envisagé une vulnérabilité IDOR
sur les fonctionnalités de compte de Manger Bouger.

Cette investigation a été limitée par la maintenance
de certaines fonctionnalités et l'ambiguïté du
périmètre concernant `www.mangerbouger.fr`.

Aucun accès non autorisé à une ressource
appartenant à un autre utilisateur n'a été démontré.

**Statut : investigation non concluante.**

## 3. CSRF et sessions Keycloak — CWE-352

Nous avons étudié l'authentification sur
`sso.mangerbouger.fr`.

Les tests ont porté sur les requêtes HTTP,
les paramètres de session et les redirections.

Une requête modifiée avec un `session_code`
invalide a produit une réponse HTTP 302.

Nous avons également observé des comportements
différents entre une session Edge existante
et une fenêtre InPrivate.

Aucun scénario CSRF complet n'a été exécuté.

**Statut : aucune vulnérabilité CSRF confirmée.**

## Signalements YesWeHack

**Nombre de vulnérabilités confirmées : 0**

**Nombre de signalements soumis : 0**

Aucun rapport de vulnérabilité n'a été envoyé
sur YesWeHack, faute de preuve d'exploitation.

## Documentation

Les investigations sont détaillées dans :

- `targets/www.tabac-info-service.fr/notes/`
- `targets/www.tabac-info-service.fr/journal.md`
- `targets/mangerbouger.fr/notes/`
- `targets/mangerbouger.fr/journal.md`
- `targets/mangerbouger.fr/evidences/`

Le rapport universitaire constitue le livrable
principal du projet.

Les scripts du dossier `scripts/` n'ont pas
été exécutés sur les cibles du programme.

## Conclusion

L'absence de vulnérabilité confirmée ne signifie
pas que les applications sont exemptes de failles.

Nos conclusions concernent uniquement les
fonctionnalités et scénarios effectivement étudiés.

Ce projet nous a permis de mettre en pratique
une méthodologie de Bug Bounty et de documenter
nos observations de manière rigoureuse.
