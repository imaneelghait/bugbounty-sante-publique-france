# Fiche cible — www.tabac-info-service.fr

- **Programme** : Santé publique France (YesWeHack)
- **Dans le périmètre** : oui, hôte listé tel quel (vérifié le 2026-11-06)
- **Débit autorisé** : 30 req/s maximum, pas d'outils automatisés
- **En-tête d'identification exigé** : `YWHSPF` ajouté à la fin du `User-Agent`
- **Classe de vulnérabilité choisie** : XSS réfléchie

## Technologies

| Couche | Détail |
|---|---|
| Serveur | Apache/Debian (observé le 2026-11-09) |
| Framework / CMS | Ibexa CMS, PHP/Symfony |
| Cache / reverse proxy | Varnish (observé) |
| CDN / WAF | Non confirmé |
| Auth | Aucun compte utilisateur sur cet hôte |
| Services tiers | Google Maps (hors périmètre), TrustCommander (consentement cookies) |

## Comptes de test

Aucune création de compte sur `www.tabac-info-service.fr` : l'inscription « Mois sans tabac » mène à `mois-sans-tabac.tabac-info-service.fr` (hors périmètre), l'accompagnement passe par une application mobile (hors périmètre). Une XSS réfléchie dans une recherche ne nécessite pas de compte.

## Surface d'attaque

| Fonctionnalité | Intérêt | Testée | Notes |
|---|---|---|---|
| Recherche principale (`/resultats-de-recherche`, `search_form[search_text]`, GET) | élevé | ☑ | `< > " '` échappés — pas de XSS démontrée (2026-11-02) |
| Autocomplétion des articles (`/autocomplete/article/<texte>`) | élevé | ☑ | Testée le 2026-11-09 avec Burp : entrée non reflétée, réponse JSON, aucune XSS démontrée. Autocomplétion des questions non testée. |
| Recherche de tabacologue (`/je-trouve-un-tabacologue`) | élevé | ☐ | champ adresse + liste de régions ; carte Google Maps hors périmètre |
| Inscription « Mois sans tabac » | — | ☐ | redirige hors périmètre, non testé |
| Espace Pro | — | ☐ | réservé aux professionnels, non testé |

## Pistes à creuser

- XSS réfléchie sur l'autocomplétion de la recherche.
- XSS réfléchie sur la recherche de tabacologue (champ adresse).
- Vérifier d'autres paramètres reflétés dans l'URL (messages, filtres, pagination).

## Écarté (et pourquoi)

- Moteur de recherche principal : caractères spéciaux échappés, testé le 2026-11-02. À ne réexaminer que si un contexte d'injection différent apparaît.
