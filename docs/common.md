# Conventions communes — tous les repos

> Règles communes à tous les repos, quel que soit le langage.

## Code
- Pas de commentaire explicatif dans le code : l'explication va dans la PR ou la discussion. Un point vraiment
  non évident tient en une ligne courte.
- Pas de solution over-engineered pour un problème simple.
- Après chaque série de modifications, lancer le build (et le type-check côté front) et corriger les erreurs
  avant de continuer.
- Un fichier qui n'a plus aucune référence se supprime dans la foulée.

## Packages Krosoft
- Les applications consomment les versions **publiées** des packages Krosoft (npm, NuGet), jamais les sources
  locales : un fix dans un package n'a d'effet qu'après publication par la CI puis bump dans l'app.

## Dépendances (Renovate)
- `renovate.json` à la racine, qui étend le preset partagé de `krosoft-dev/krosoft-tooling-renovate` adapté à la
  stack (`:react`, `:dotnet`...). Seul `assignees` est propre au repo ; une règle utile à plusieurs repos va dans
  le preset, pas en local.
- Les montées de version passent par les PR Renovate (packages Krosoft toujours sur la dernière version). Bump
  manuel seulement quand un fix publié est attendu tout de suite.

## Documentation (`docs/`)
```
docs/
├── conventions/                   # copies des conventions (repos .NET ; les fronts importent depuis node_modules)
├── features/{feature}.md          # fonctionnement d'une fonctionnalité livrée
├── plans/{chantier}/              # README.md (objectif, découpage) + lot-{n}-{slug}.md
├── adrs/{projet}-{nnn}-{slug}.md  # décisions d'architecture
├── release-notes/{fr,en}.md       # notes de version destinées aux utilisateurs
└── roadmap.md                     # ce qui reste à faire
```
- **features** : doc durable (règles métier, flux, choix techniques), mise à jour dans la même PR que le code.
- **plans** : doc de travail d'un chantier en cours, un lot par fichier. Une fois le chantier livré, ce qui doit
  durer passe dans `features/` ou un ADR, puis le plan est supprimé.
- **adrs** : une décision par fichier, numérotée, avec statut (Proposé, Accepté, Rejeté, Déprécié par...).
- **release-notes** : une entrée par livraison visible, la plus récente en haut, titre `## yyyy-MM-dd HH:mm - Titre`,
  texte orienté utilisateur ; `fr.md` et `en.md` mis à jour ensemble.
- **roadmap** : ne liste que le restant ; un élément livré en sort (et passe dans les release notes).

## CLAUDE.md des repos
Sections dans cet ordre :
1. Description du projet : titre et contexte en quelques lignes.
2. `## Conventions` : les imports des conventions partagées, juste après la description.
3. `## Exceptions aux conventions` : uniquement les écarts aux conventions importées, ou « Aucune. ».
4. Les sections propres au repo qui ne dérogent à aucune convention (à savoir, docs, notes de version...).
