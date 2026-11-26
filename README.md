# krosoft-tooling-conventions

[![forthebadge](https://forthebadge.com/badges/built-with-love.svg)](https://forthebadge.com)

Krosoft shared code conventions for Claude Code.

Conventions de code génériques, au format Markdown, à importer dans le `CLAUDE.md` des repos consommateurs.
Ce qui est propre à un repo reste dans son propre `CLAUDE.md`.

## Fichiers

| Fichier                  | Contenu                                                                  |
| ------------------------ | ------------------------------------------------------------------------ |
| `docs/common.md`         | Règles communes à tous les repos, quel que soit le langage (Renovate...) |
| `docs/react.md`          | Frontends React sur `@krosoft/react`                                     |
| `docs/react-supabase.md` | Complément de `react.md` pour les frontends adossés à Supabase           |
| `docs/dotnet.md`         | Socle des repos .NET (repo uniquement, hors package)                     |
| `docs/dotnet-webapi.md`  | Complément de `dotnet.md` pour les APIs (repo uniquement, hors package)  |

## Installation

```bash
npm i -D @krosoft/tooling-conventions
```

## Usage

Dans le `CLAUDE.md` du repo, importer les fichiers utiles :

```markdown
@node_modules/@krosoft/tooling-conventions/docs/common.md
@node_modules/@krosoft/tooling-conventions/docs/react.md
@node_modules/@krosoft/tooling-conventions/docs/react-supabase.md
```

Les mises à jour arrivent par Renovate, sans copie dans le repo.

## Repos .NET (sans `node_modules`)

`docs/dotnet.md` et `docs/dotnet-webapi.md` ne sont pas publiés dans le package npm. Les copier, avec
`docs/common.md`, depuis ce repo GitHub dans `docs/conventions/` de chaque repo .NET et les importer depuis le
`CLAUDE.md` :

```markdown
@docs/conventions/common.md
@docs/conventions/dotnet.md
@docs/conventions/dotnet-webapi.md
```

La copie se met à jour à la main depuis ce repo.

## Contribuer

Toute modification passe par une PR sur ce repo ; la publication sur npm est faite par la CI au merge sur `main`.
