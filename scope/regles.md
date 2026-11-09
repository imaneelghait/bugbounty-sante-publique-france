# Règles du programme — Santé publique France

- **Plateforme** : YesWeHack (programme public « Santé Publique France Bug Bounty Program »)
- **URL de la policy** : https://yeswehack.com/programs/sante-publique-france-bugbounty-public-program
- **Fiche consultée le** : 06/11/2026 (dernière mise à jour de la fiche : 09/03/2026)
- **Safe harbor** : non mentionné dans la fiche — aucune clause explicite de protection juridique ; on reste donc strictement dans les règles
- **Débit maximum** : 30 requêtes/seconde (tests en production)
- **En-tête d'identification exigé** : ajouter `YWHSPF` à la fin du `User-Agent`
- **Scanners automatisés** : interdits — « refrain from using automated tools », éviter tout test pouvant dégrader ou interrompre le service

## Techniques interdites

- [x] DoS / stress test — interdit explicitement, ainsi que toute interférence avec l'infrastructure ou les équipements réseau
- [x] Brute force — rien qui dégrade le service ; absence de limitation de débit ou brute force non qualifiants
- [x] Social engineering / phishing — ingénierie sociale du personnel non qualifiante
- [x] Spam (SMS, e-mail) — capacité à spammer les utilisateurs non qualifiante
- [x] Tests sur des comptes tiers — interdiction de divulguer, manipuler ou détruire des données d'utilisateurs
- [x] Autre :
  - modifier les données de production : les tests d'intégrité se font sur l'environnement PPROD (adresse non fournie par la fiche) ;
  - impacter l'activité d'autres utilisateurs : une XSS stockée utilise `console.log()`, jamais `alert()`, `confirm()`, `prompt()` ;
  - divulguer une vulnérabilité, même partiellement ;
  - aller au-delà des preuves autorisées : RCE → `id`, `whoami`, `hostname`, `ifconfig` ; SQLi → version et/ou schéma de la base ; SSRF → contenu de page ou comportement spécifique.

## Vulnérabilités hors récompense (known issues)

À relire AVANT chaque soumission :

- Liens cassés / détournement de comptes de réseaux sociaux
- Tabnabbing
- Attributs de cookies manquants
- Injection de contenu ou de texte
- Clickjacking / UI redressing
- Déni de service (DoS)
- CVE récentes (moins de 30 jours après le correctif)
- CVE sans vulnérabilité exploitable ni PoC
- Ports ou services ouverts sans vulnérabilité exploitable ni PoC
- Ingénierie sociale du personnel ou des prestataires
- Attribut `autocomplete` sur les formulaires
- Vulnérabilités touchant des navigateurs ou plateformes obsolètes
- Self-XSS ou XSS ne pouvant pas toucher d'autres utilisateurs
- Faille hypothétique ou bonne pratique
