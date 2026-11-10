# Analyse XSS réfléchie — Filtres et pagination

## 1. Objectif

Vérifier si les paramètres GET `theme` et `page` de la page Questions / Réponses permettent une injection XSS réfléchie.

**Cible :** `https://www.tabac-info-service.fr/questions-reponses`

**Date :** 10/11/2026

**Outil :** Burp Suite Community — Repeater

**Méthode :** Tests manuels, sans scanner automatisé.

## 2. Paramètre `theme`

### Test 1 — Valeur normale

```http
GET /questions-reponses?theme=2481538 HTTP/2
```

**Résultat :** HTTP 200 OK.

Le thème « Dépendance » est sélectionné et les questions correspondantes sont affichées.

### Test 2 — Marqueur de réflexion

```http
GET /questions-reponses?theme=TESTXSS123 HTTP/2
```

**Résultat :** HTTP 200 OK.

Le marqueur apparaît dans un attribut HTML :

```html
<div class="Controls-archives" data-target="/questions-reponses?theme=TESTXSS123&amp;archive=true" data-archives>
```

**Observation :** la valeur du paramètre `theme` est réinjectée dans la réponse HTML.

### Test 3 — Caractères spéciaux

```http
GET /questions-reponses?theme=TESTXSS123%22%3E HTTP/2
```

**Résultat :** HTTP 500 Internal Server Error.

Le serveur retourne une page d'erreur générique. Aucun contenu HTML contenant le marqueur n'est renvoyé dans cette réponse.

**Observation :** cette entrée provoque une erreur côté serveur. La cause exacte n'a pas été déterminée.

## 3. Paramètre `page`

### Test 4 — Pagination normale

```http
GET /questions-reponses?theme=2481538&page=2 HTTP/2
```

**Résultat :** HTTP 200 OK.

La deuxième page des résultats est affichée.

Le paramètre `page` est présent dans les liens HTML de navigation et de filtres :

```html
<a href="/questions-reponses?page=2&amp;theme=2481224">
```

### Test 5 — Valeur non numérique

```http
GET /questions-reponses?theme=2481538&page=TESTPAGE123 HTTP/2
```

**Résultat :** HTTP 500 Internal Server Error.

Le serveur retourne une page d'erreur générique. Le marqueur n'apparaît pas dans la réponse fournie.

**Observation :** une valeur non numérique du paramètre `page` provoque une erreur interne dans ce test.

## 4. Analyse

- Le paramètre `theme` reflète une valeur alphanumérique dans un attribut HTML.
- Le paramètre `page` est utilisé dans les liens de navigation avec une valeur numérique valide.
- Certaines entrées inattendues entraînent des réponses HTTP 500.
- Aucun code JavaScript injecté n'a été exécuté.
- Aucune XSS réfléchie n'a été démontrée.

Les réponses HTTP 500 ne permettent pas, à elles seules, de conclure à une vulnérabilité exploitable.

## 5. Conclusion

**Statut : XSS réfléchie non démontrée sur les paramètres testés.**

Les observations sont conservées pour le rapport final. Les tests sur les valeurs provoquant des erreurs HTTP 500 n'ont pas été poursuivis afin de limiter les risques de perturbation du service.

## 6. Preuves

Réponses HTTP conservées dans Burp Suite, datées du 10/11/2026.
