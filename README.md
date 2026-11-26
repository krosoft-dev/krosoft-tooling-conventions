# krosoft-tooling-conventions

[![forthebadge](https://forthebadge.com/badges/built-with-love.svg)](https://forthebadge.com)

Krosoft shared code conventions for Claude Code.

Conventions de code génériques, au format Markdown, à importer dans le `CLAUDE.md` des repos consommateurs.
Ce qui est propre à un repo reste dans son propre `CLAUDE.md`.

## Fichiers

| Fichier       | Contenu                                                                  |
| ------------- | ------------------------------------------------------------------------ |
| `react.md`    | Frontends React sur `@krosoft/react`                                     |
| `supabase.md` | Complément de `react.md` pour les frontends adossés à Supabase           |
| `dotnet.md`   | Backends .NET sur `Krosoft.Extensions` (vertical slices, CQRS, tests)    |

## Installation

```bash
npm i -D @krosoft/tooling-conventions
```

## Usage

Dans le `CLAUDE.md` du repo, importer les fichiers utiles :

```markdown
@node_modules/@krosoft/tooling-conventions/react.md
@node_modules/@krosoft/tooling-conventions/supabase.md
```

Les mises à jour arrivent par Renovate, sans copie dans le repo.

## Repos .NET (sans `node_modules`)

Copier `dotnet.md` dans `docs/conventions/` du repo et l'importer depuis le `CLAUDE.md` :

```markdown
@docs/conventions/dotnet.md
```

La copie se met à jour à la main depuis ce package.

## Contribuer

Toute modification passe par une PR sur ce repo ; la publication sur npm est faite par la CI au merge sur `main`.
