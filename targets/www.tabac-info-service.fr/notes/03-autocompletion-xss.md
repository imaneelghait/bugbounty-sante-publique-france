# 03 — Analyse : XSS réfléchie sur l'autocomplétion

- **Date :** 2026-11-09
- **Cible :** www.tabac-info-service.fr
- **Fonctionnalité :** autocomplétion du moteur de recherche
- **Outil :** Burp Suite (Proxy + Repeater), User-Agent `YWHSPF`
- **Statut :** aucune vulnérabilité démontrée

## Objectif

Vérifier si le texte saisi dans la recherche, envoyé à l'endpoint d'autocomplétion, est renvoyé sans échappement (XSS réfléchie).

## Point d'entrée identifié

- Endpoint : `GET /autocomplete/article/<texte>`
- Le texte saisi est placé **dans le chemin de l'URL**.
- Réponse : `Content-Type: application/json`.

## Tests réalisés

| Entrée | Réponse | Observation |
|---|---|---|
| `tabac` | `[{"url":"/article/mois-sans-tabac","text":"Mois sans Tabac"}, …]` | Le serveur renvoie des titres d'articles correspondants, **pas le texte saisi** |
| `zzqxyw123` (aucun résultat) | `[]` | Liste vide, texte non renvoyé |
| `<zzz123>` (via Repeater) | `[]`, statut 200, `Content-Type: application/json` | Caractères spéciaux non renvoyés, pas de changement de contexte |

## Résultat

Le texte saisi n'apparaît jamais dans la réponse, que la recherche donne des résultats ou non. La réponse est systématiquement du JSON, que le navigateur n'interprète pas comme du HTML.

## Conclusion

L'endpoint d'autocomplétion n'est pas vulnérable à une XSS réfléchie : absence de réflexion de l'entrée et réponse en `application/json`.

## À vérifier plus tard

- L'endpoint équivalent `/autocomplete/question/<texte>` (le code HTML mentionne `data-search-autocomplete="question"`).
- Le rendu des suggestions côté JavaScript (fichier `main.*.js`), pour une éventuelle DOM XSS, les titres venant toutefois du serveur et non de l'utilisateur.
