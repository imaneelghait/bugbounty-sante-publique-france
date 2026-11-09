# Journal de bord — www.tabac-info-service.fr

Une entrée par session. Sert de preuve de bonne foi et de mémoire de travail.

## 2026-11-02 — horaire non noté (Melissa)

**Périmètre du jour :** `www.tabac-info-service.fr`, moteur de recherche principal

### Fait
- Identification du paramètre de recherche : `search_form[search_text]`, méthode `GET`, endpoint `/resultats-de-recherche`.
- Test 1 : `test<bonjour>` → renvoyé `test&lt;bonjour&gt;`.
- Test 2 : `test"bonjour'fin` → renvoyé `test&quot;bonjour&#039;fin`.
- Analyse avec les outils de développement de Chrome (Network → Document → Response).

### Observations
- `< > " '` échappés dans l'attribut `value` du champ et dans le titre des résultats.
- Test fait sans Burp : présence du User-Agent `YWHSPF` non vérifiée. Corrigé à partir du 2026-11-06 (règle Burp).

### Impasses (ne pas refaire)
- Ces deux contextes du moteur de recherche principal : pas de XSS démontrée.

### Prochaine étape
- Étudier l'autocomplétion de la recherche.

## 2026-11-06 — ≈ 22:20 à 22:55 (Imane)

**Périmètre du jour :** `www.tabac-info-service.fr` (navigation, aucune charge de test)

### Fait
- Mise en place de la règle Burp ajoutant `YWHSPF` au User-Agent ; vérifiée sur `/?verif=2a` (version « Auto-modified request »).
- Recherche d'une création de compte sur le site.
- Navigation dans le menu « Je me fais accompagner ».

### Observations
- Aucune création de compte sur `www.tabac-info-service.fr`.
- Le bouton « Je participe » (Mois sans tabac) mène à `mois-sans-tabac.tabac-info-service.fr/inscription` : **hors périmètre**, formulaire non rempli.
- « Je télécharge l'application » : application mobile, hors périmètre.
- Point d'entrée repéré : `/je-trouve-un-tabacologue` (champ « Adresse, ville ou département » + liste de régions).
- Requêtes vers `maps.googleapis.com` (carte Google Maps) : service tiers, hors périmètre.

### Impasses (ne pas refaire)
- Comptes de test sur cet hôte : inexistants dans le périmètre.

### Prochaine étape
- Reconnaissance applicative : technologies, fichiers JavaScript, `robots.txt`, `sitemap.xml`, autocomplétion.
