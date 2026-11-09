# Analyse 01 : recherche de XSS réfléchie

- **Date :** 2026-11-02
- **Auteure :** Melissa GUERAOUNI
- **Cible :** www.tabac-info-service.fr
- **Fonctionnalité :** moteur de recherche
- **Statut :** aucune vulnérabilité démontrée

## Objectif

Vérifier si les caractères spéciaux saisis dans le moteur de recherche sont correctement échappés dans la réponse HTML.

## Méthodologie

Analyse manuelle avec les outils de développement Chrome (Network → Document → Response).

- Paramètre identifié : `search_form[search_text]`
- Méthode HTTP : `GET`
- Endpoint : `/resultats-de-recherche`

## Tests réalisés

### Test 1 : caractères HTML
- Entrée : `test<bonjour>`
- Réponse dans le champ de recherche : `test&lt;bonjour&gt;`

### Test 2 : guillemets
- Entrée : `test"bonjour'fin`
- Réponse dans le champ de recherche : `test&quot;bonjour&#039;fin`

## Résultats

Les caractères `<`, `>`, `"` et `'` sont échappés dans les deux emplacements observés : l'attribut `value` du champ de recherche et le titre des résultats.

## Conclusion

Aucune vulnérabilité XSS réfléchie n'a été démontrée avec ces tests. Cette conclusion est limitée aux entrées et contextes HTML examinés.

## Remarque

Test réalisé sans Burp : la présence du User-Agent `YWHSPF` n'a pas été vérifiée. Les tests suivants passent par Burp avec la règle `YWHSPF`.

## Prochaine étape

Étudier manuellement le fonctionnement de l'autocomplétion du moteur de recherche, dans le respect du périmètre et des règles du programme.
