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
