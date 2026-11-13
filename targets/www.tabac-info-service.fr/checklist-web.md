# Checklist web — www.tabac-info-service.fr

Copie de `docs/02-checklist-web.md`.
Règle : une case non cochée = zone non testée, pas zone saine.

> **Contexte.** Le projet a porté sur **une classe** (XSS réfléchie) sur une cible **sans authentification**. Les sections Authentification, Session et une grande partie des injections restent donc non cochées : non applicables ou hors de la classe choisie. Détails dans les notes 02 à 05.

## Reconnaissance applicative
- [x] Technologies identifiées (serveur, framework, CDN, WAF) — Apache/Debian, Ibexa (Symfony), cache Varnish ; WAF non identifié
- [ ] Fichiers JS téléchargés et lus — non approfondi
- [ ] `robots.txt`, `sitemap.xml`, `.well-known/`, `/swagger`, `/graphql`, `/actuator` — non testé
- [x] Sous-domaines et environnements annexes — identifiés : `mois-sans-tabac.`, `pro.` (hors périmètre)
- [ ] Répertoires exposés : `.git/`, `.env`, `backup/`, `.DS_Store`, dumps SQL — non testé
- [x] Messages d'erreur verbeux, stack traces, pages de debug — erreurs 500 observées, **génériques, sans fuite d'information**

## Authentification
*Non applicable : aucun compte utilisateur sur `www.tabac-info-service.fr`.*
- [ ] Énumération d'utilisateurs
- [ ] Politique de mot de passe, verrouillage, rate limiting
- [ ] Reset de mot de passe
- [ ] MFA
- [ ] OAuth / SSO
- [ ] JWT

## Session
*Non applicable : pas de session authentifiée sur cet hôte.*
- [ ] Attributs des cookies
- [ ] Session non invalidée
- [ ] Session fixation
- [ ] Sessions concurrentes

## Contrôle d'accès
*Non applicable sur cette cible (pas de compte, contenu entièrement public). Classe IDOR envisagée sur la cible secondaire — voir `targets/mangerbouger.fr/notes/00-observation-plan-b.md`.*
- [ ] IDOR
- [ ] Élévation verticale
- [ ] Élévation horizontale
- [ ] Contrôle uniquement côté client
- [ ] Changement de méthode HTTP
- [ ] Contournement de chemin

## Injections
*Hors de la classe choisie — non testé.*
- [ ] SQL / NoSQL
- [ ] Injection de commandes OS
- [ ] SSTI
- [ ] LDAP, XPath, CRLF
- [ ] Désérialisation

## XSS et injections côté client
- [x] Réfléchie, stockée, DOM-based — **XSS réfléchie testée sur 5 points d'entrée** (notes 02-04) ; stockée et DOM-based non testées
- [ ] Contournement de la CSP / absence de CSP — non testé
- [ ] Upload de fichier rendu en HTML ou SVG — pas de fonctionnalité d'upload testée
- [ ] `target=_blank` sans `noopener`, open redirect — non testé

## Logique métier
*Hors de la classe choisie — non testé.*
- [ ] Manipulation prix / quantité / remise
- [ ] Étapes de tunnel sautées
- [ ] Valeurs limites
- [ ] Race conditions
- [ ] Absence de rate limit sur action coûteuse

## Côté serveur
*Hors de la classe choisie — non testé.*
- [ ] SSRF
- [ ] Inclusion / traversée de fichiers
- [ ] XXE
- [ ] Upload
- [ ] Request smuggling, cache poisoning

## Configuration
- [ ] CORS permissif — non testé
- [ ] En-têtes de sécurité — observés (HSTS présent) mais non retenus (non récompensés par le programme)
- [ ] Prise de contrôle de sous-domaine — non testé
- [ ] Identifiants par défaut — non testé
- [ ] Versions vulnérables connues (CVE) — non testé

## Avant de rapporter
*Non applicable : aucune vulnérabilité retenue.*
- [ ] Reproduit deux fois
- [ ] Impact métier formulé
- [ ] Preuves caviardées et horodatées
- [ ] Vérifié hors de la liste des exclusions
