# 04 — Analyse : XSS réfléchie sur les filtres Questions / Réponses

- **Date :** 2026-11-10
- **Auteures :** Melissa GUERAOUNI (tests initiaux) et Imane EL GHAIT (caractérisation)
- **Cible :** www.tabac-info-service.fr
- **Page :** `/questions-reponses`
- **Paramètres :** `theme` et `page` (GET)
- **Outil :** Burp Suite (Repeater), User-Agent `YWHSPF`
- **Statut :** aucune vulnérabilité démontrée

## Objectif

Vérifier si les paramètres de filtre `theme` et `page` de la page Questions / Réponses permettent une XSS réfléchie.

## Point d'entrée et contexte de réflexion

La valeur de `theme` est reflétée dans un attribut HTML, à l'intérieur d'une URL :

```html
<div class="Controls-archives" data-target="/questions-reponses?theme=VALEUR&amp;archive=true" data-archives>
```

La valeur est donc placée dans un attribut délimité par des guillemets doubles. Pour une XSS, il faudrait soit injecter un `"` pour sortir de l'attribut, soit injecter `<` pour ouvrir une balise.

## Tests réalisés — paramètre `theme`

| Entrée | Statut | Reflété comme | Interprétation |
|---|---|---|---|
| `2481538` (thème valide) | 200 | — | Fonctionnement normal, thème « Dépendance » |
| `TESTXSS123` | 200 | `TESTXSS123` | Valeur alphanumérique reflétée |
| `aaa<bbb` | 200 | `aaa%3Cbbb` | `<` encodé en `%3C` |
| `aaa>bbb` | 200 | `aaa%3Ebbb` | `>` encodé en `%3E` |
| `aaa'bbb` | 200 | `aaa%27bbb` | `'` encodé en `%27` |
| `aaa"bbb` | 500 | — | Erreur serveur, aucune réflexion |

## Tests réalisés — paramètre `page`

| Entrée | Statut | Interprétation |
|---|---|---|
| `2` | 200 | Pagination normale |
| `TESTPAGE123` (non numérique) | 500 | Erreur serveur, aucune réflexion |

## Analyse

- Le paramètre `theme` est reflété, mais le serveur **encode en URL** les caractères HTML dangereux (`<` → `%3C`, `>` → `%3E`, `'` → `%27`). Ils ne peuvent donc pas former de balise ni d'attribut.
- Le seul caractère permettant de sortir de l'attribut (`"`) provoque une **erreur 500** : le serveur n'atteint pas le rendu de la page, et la valeur n'est pas reflétée.
- Les pages d'erreur 500 sont **génériques** (titre « An Error Occurred », aucune trace technique, aucun chemin de fichier exposé). Elles ne reflètent pas l'entrée.
- Aucun code JavaScript n'a pu être injecté ni exécuté.

## Conclusion

**XSS réfléchie non démontrée sur `theme` et `page`.** La réflexion existe mais est neutralisée par l'encodage URL, et la rupture de l'attribut est impossible (erreur 500). Les erreurs 500 observées, génériques et sans fuite d'information, ne constituent pas une vulnérabilité récompensable selon la politique du programme.

## Précautions

Tests limités à quelques requêtes manuelles dans Repeater. Les entrées provoquant une erreur 500 n'ont pas été répétées, afin de ne pas perturber le service.

## Preuves

Requêtes et réponses conservées dans Burp Suite, datées du 10/11/2026.
