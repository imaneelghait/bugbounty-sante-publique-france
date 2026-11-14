
# Bug Bounty — Santé publique France

**Projet académique — Master 2 Cybersécurité**  
Université de Caen Normandie / ENSICAEN

**Binôme (groupe 4) :**
- Imane EL GHAIT
- Melissa GUERAOUNI

**Période :** Novembre 2026  
**Plateforme :** YesWeHack  
**Programme :** [Santé publique France — Bug Bounty](https://yeswehack.com/programs/sante-publique-france-bugbounty-public-program)

---

## 1. Présentation du projet

Ce dépôt regroupe notre travail réalisé dans le cadre
d'un projet universitaire de Bug Bounty.

L'objectif était de découvrir la méthodologie d'une
recherche de vulnérabilités sur un programme réel,
en respectant les règles définies par Santé publique France.

Nous avons travaillé en binôme, avec une approche
principalement manuelle, en utilisant Burp Suite
pour observer et analyser les échanges HTTP.

Ce dépôt constitue notre carnet de travail :
méthodologie, journal des tests, analyses techniques
et preuves.

Le rapport universitaire constitue le livrable principal.

## 2. Périmètre et règles

Le programme étudié est celui de Santé publique France
sur la plateforme YesWeHack.

Les hôtes principalement étudiés sont :

| Hôte | Utilisation |
|---|---|
| `www.tabac-info-service.fr` | Cible principale : XSS réfléchie |
| `mangerbouger.fr` | Reconnaissance et étude de la piste IDOR |
| `sso.mangerbouger.fr` | Analyse Keycloak et gestion des sessions |

Le statut de `www.mangerbouger.fr` n'étant pas
explicitement confirmé dans le périmètre,
aucun test actif n'a été réalisé sur cet hôte.

### Règles respectées

- Respect des hôtes autorisés.
- Identification des requêtes avec `YWHSPF`
  à la fin du User-Agent.
- Tests manuels sans scanner automatisé.
- Respect de la limite de 30 requêtes/seconde.
- Aucun brute force ni déni de service.
- Aucune modification de données de production.
- Aucun accès volontaire à des comptes tiers.

Les premières observations réalisées avant la
configuration complète de Burp sont identifiées
comme telles dans le journal de bord.

Les règles détaillées sont disponibles dans
[scope/regles.md](scope/regles.md).

## 3. Méthodologie

Notre démarche s'est organisée en plusieurs étapes :

1. Lecture du programme et de ses règles.
2. Identification du périmètre autorisé.
3. Préparation de l'environnement de test.
4. Reconnaissance des fonctionnalités.
5. Identification des points d'entrée.
6. Formulation d'hypothèses de vulnérabilités.
7. Tests manuels avec Burp Suite.
8. Analyse des réponses HTTP.
9. Documentation des observations et des limites.
10. Synthèse des résultats.

### Outils utilisés

- Microsoft Edge
- Burp Suite Community
- FoxyProxy
- Outils de développement du navigateur
- Git et GitHub

Les scripts fournis dans le dépôt à titre
pédagogique n'ont pas été exécutés sur les
applications du programme.

## 4. Investigations techniques

### 4.1 XSS réfléchie — CWE-79

**Cible :** `www.tabac-info-service.fr`

Nous avons recherché des vulnérabilités XSS réfléchies
sur plusieurs points d'entrée.

| Point d'entrée | Observation |
|---|---|
| Recherche principale | Caractères spéciaux échappés dans les réponses HTML |
| Autocomplétion | Entrée testée non reflétée, réponse JSON |
| Filtre `theme` | Encodage des caractères, erreur HTTP 500 pour certaines valeurs |
| Filtre `page` | Erreur HTTP 500 avec une valeur non numérique |
| Recherche de tabacologue | Fonctionnalité reposant sur Google Maps, service tiers hors périmètre |

Les erreurs HTTP 500 observées ne contenaient pas
de divulgation d'informations sensibles constatée.

**Résultat : aucune XSS exploitable démontrée
sur les points d'entrée testés.**

Les analyses détaillées sont disponibles dans :

- [Recherche principale](targets/www.tabac-info-service.fr/notes/02-recherche-principale-xss.md)
- [Autocomplétion](targets/www.tabac-info-service.fr/notes/03-autocompletion-xss.md)
- [Filtres Questions/Réponses](targets/www.tabac-info-service.fr/notes/04-filtres-questions-reponses-xss.md)

### 4.2 IDOR — CWE-639

**Cible étudiée :** Manger Bouger

Nous avons envisagé une vulnérabilité de type IDOR,
correspondant à un défaut de contrôle d'accès
sur des ressources appartenant à un utilisateur.

Lors de la reconnaissance, nous avons identifié
un espace compte et un mécanisme d'authentification
reposant sur Keycloak.

Cependant, plusieurs limites ont empêché
d'approfondir cette investigation :

- Maintenance de certaines fonctionnalités.
- Ambiguïté concernant le périmètre `www.mangerbouger.fr`.
- Absence d'identifiant d'objet directement
  manipulable dans l'URL de la page observée.

**Résultat : aucune vulnérabilité IDOR démontrée.**

Cette piste reste non concluante en raison
des limites de couverture.

[Consulter les observations Manger Bouger](targets/mangerbouger.fr/notes/00-observation-plan-b.md)

### 4.3 Authentification Keycloak et CSRF — CWE-352

**Cible :** `sso.mangerbouger.fr`

Nous avons étudié le parcours d'authentification
Keycloak utilisant OpenID Connect.

Les investigations ont porté sur :

- Les requêtes d'authentification.
- Les redirections HTTP 302.
- Les paramètres de session.
- Les cookies associés à l'authentification.
- Le comportement du serveur après modification
  du paramètre `session_code`.
- La comparaison de différents contextes
  de navigation.

Dans Burp Repeater, nous avons envoyé une requête
avec un `session_code` invalide et des identifiants
fictifs.

Le serveur a retourné une réponse HTTP 302
avec une redirection vers le parcours
d'authentification et un nouveau cookie de session.

Nous avons également observé deux comportements
différents selon le contexte de navigation :

- Session déjà connectée :
  « Vous êtes déjà connecté ».
- Fenêtre InPrivate :
  `cookieNotFoundMessage`.

Ces observations montrent l'importance du
contexte de session, mais ne permettent pas
de conclure à une vulnérabilité.

Aucun scénario CSRF complet n'a été exécuté.

**Résultat : aucune vulnérabilité CSRF démontrée.**

[Consulter l'analyse Keycloak](targets/mangerbouger.fr/notes/01-analyse-keycloak.md)

## 5. Synthèse des résultats

| Classe | CWE | Résultat |
|---|---|---|
| XSS réfléchie | CWE-79 | Non démontrée sur les points testés |
| IDOR | CWE-639 | Non démontrée, tests limités |
| CSRF | CWE-352 | Non démontrée, investigation exploratoire |

**Aucune vulnérabilité exploitable n'a été confirmée
au cours de ce projet.**

Ce résultat ne signifie pas que les applications
sont exemptes de vulnérabilités.

Nos conclusions concernent uniquement les
fonctionnalités et scénarios effectivement étudiés.

Aucun signalement de vulnérabilité n'a été
effectué sur YesWeHack.

## 6. Organisation du dépôt

```text
.
├── README.md
├── CONTRIBUTING.md
├── scope/
│   ├── in-scope.txt
│   ├── out-of-scope.txt
│   └── regles.md
├── docs/
│   └── Méthodologie et documentation
├── targets/
│   ├── www.tabac-info-service.fr/
│   │   ├── fiche.md
│   │   ├── journal.md
│   │   ├── checklist-web.md
│   │   ├── notes/
│   │   └── evidences/
│   └── mangerbouger.fr/
│       ├── fiche.md
│       ├── notes/
│       └── evidences/
├── findings/
│   ├── STATUT.md
│   ├── 1-brouillon/
│   ├── 2-soumis/
│   └── 3-resolu/
├── scripts/
└── templates/
```

Les dossiers `docs/`, `scripts/` et `templates/`
contiennent notamment des ressources pédagogiques
fournies par l'encadrant.

Le dossier `findings/1-brouillon/` contient
un exemple pédagogique fictif, qui ne correspond
pas à une vulnérabilité découverte pendant ce projet.

## 7. Difficultés rencontrées

Plusieurs difficultés ont été rencontrées :

- Configuration de Burp Suite et du User-Agent
  d'identification.
- Interprétation des requêtes et réponses HTTP.
- Respect des restrictions du périmètre.
- Maintenance de certaines fonctionnalités.
- Analyse des paramètres et cookies Keycloak.
- Coordination du travail en binôme.
- Documentation des tests et des résultats.

Ces difficultés nous ont permis d'améliorer
notre méthode de travail et notre compréhension
des applications web.

## 8. Conclusion

Ce projet nous a permis de mettre en pratique
une démarche de Bug Bounty sur un programme réel.

Nous avons étudié trois classes de vulnérabilités :
XSS, IDOR et CSRF.

Même si aucune vulnérabilité exploitable
n'a été confirmée, nous avons appris à
formuler des hypothèses, réaliser des tests
manuels, analyser les échanges HTTP et
documenter nos observations.

Nous retenons surtout qu'une recherche de
vulnérabilités ne se mesure pas uniquement
au nombre de failles découvertes.

La rigueur méthodologique, le respect du
périmètre et la qualité de la documentation
font également partie des résultats du projet.
