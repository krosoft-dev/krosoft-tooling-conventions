# Conventions .NET — stack Krosoft.Extensions

> Source de vérité : package npm `@krosoft/tooling-conventions` (`dotnet.md`). Les repos .NET n'ayant pas de
> `node_modules`, ce fichier se copie à l'identique dans `docs/conventions/` de chaque repo backend. Toute
> modification passe par une PR sur `krosoft-tooling-conventions` puis une publication, avant d'être
> propagée. Ce qui est propre à un repo va dans son `CLAUDE.md`, jamais ici.

## Stack
- **.NET 10+** pour les nouveaux projets
- **ASP.NET Core Minimal APIs** (pattern `IEndpoint` de `Krosoft.Extensions.WebApi`)
- **MediatR** — CQRS (commands / queries / handlers), **FluentValidation**
- **Entity Framework Core** — PostgreSQL
- **Krosoft.Extensions.*** — CQRS, Data, WebApi, Testing : toujours les préférer aux solutions tierces

## Organisation en vertical slices
```
src/{Projet}.Api/Features/{Feature}/
├── {Feature}Endpoint.cs
├── {Feature}Profile.cs                      # AutoMapper si besoin
└── {Operation}/
    ├── {Feature}{Operation}Command.cs       # ou Query
    ├── {Feature}{Operation}CommandHandler.cs
    ├── {Feature}{Operation}CommandValidator.cs
    └── {Feature}{Operation}Exception.cs     # exception métier
```
- Code partagé entre features : `Features/{Feature}/Shared/`, puis `Shared/` au niveau du projet.
- Namespace = chemin de dossier : `namespace Boutique.Api.Features.Articles.Delete`.
- Types scoped à une feature en `internal`.

## Endpoints (Minimal API)
```csharp
public class ArticlesEndpoint : IEndpoint
{
    public RouteGroupBuilder DefineGroup(WebApplication app) =>
        app.MapTaggedGroup("/Articles");

    public void Register(RouteGroupBuilder group)
    {
        group.MapGet("/{articleId:guid}", ...);
        group.MapDelete("/{articleId:guid}", ...);
    }
}
```
- Aucune logique métier dans un endpoint : il construit la Command/Query et délègue à MediatR.

## Commands et Queries
```csharp
internal record ArticleDeleteCommand(Guid ArticleId) : BaseCommand;
internal record ArticlesDisponiblesQuery(Guid CategorieId) : BaseQuery<IEnumerable<ArticleDto>>;
```

## Nommage des contrats d'API (DTO)
- **Entrée HTTP** : binder directement la Command/Query (`[FromBody] CommandeCreateCommand command`,
  `[AsParameters] CommandesQuery query`) ; les ids de route s'injectent via `command with { Id = id }`.
  Pas de DTO d'entrée miroir.
- **`{Operation}Request`** uniquement quand la forme HTTP diffère réellement de la Command (ex. multipart
  `IFormFile` converti en `KrosoftFile`) : `ArticleImageUploadRequest`. Jamais `{Operation}CommandDto`.
- **`{Ressource}Dto`** (un nom, jamais un verbe) pour une sortie qui représente une ressource :
  `ArticleDetailDto`, `CommandeDto`.
- **`{Operation}ResultDto`** pour la sortie d'une action : `LoginResultDto`, `CommandeValidationResultDto`.
- Pas de `{Verbe}Dto` (ambigu entrée/sortie) ni de couple `Request`/`Response` systématique.
- Un nom court de DTO doit être unique dans une API : Swashbuckle l'utilise comme schemaId, et deux types
  homonymes dans des namespaces différents cassent `swagger.json`. Un `SwaggerTests` fonctionnel
  (GET `/swagger/v1/swagger.json` → 200) verrouille ce point.

## Données
- `IReadRepository<T>` / `IWriteRepository<T>` génériques de `Krosoft.Extensions.Data` + `IUnitOfWork`.
  Jamais de `DbContext` injecté dans un handler, jamais de repository concret.
- Configurations EF Core dans `Data/Configurations/{Entity}Configuration.cs` (`EntityTypeConfiguration<T>`).

## DI
- Extension methods par feature / module : `.AddArticles()`, `.Add{Projet}()`.

## Configuration projet
`Directory.Build.props` : `ImplicitUsings`, `Nullable` et `TreatWarningsAsErrors` activés. Un warning se
corrige, il ne se supprime pas.

## Tests
- Projets dans `tests/`, suffixés `.Tests`, répartis en `Unit/` et `Functional/`, socle dans `Core/`.
- Framework **MSTest**, assertions **NFluent**, mocks **Moq**.
- Classes de base `{Projet}BaseTest` / `{Projet}BaseApiTest` (sur `Krosoft.Extensions.Testing`),
  seed via `{Projet}SeedService`, constantes dans `{Projet}TestConstants`.
- EF Core **in-memory**, jamais de mock d'EF Core.
- Packages : `Krosoft.Extensions.Testing`, `Krosoft.Extensions.Testing.Cqrs`,
  `Krosoft.Extensions.Testing.WebApi`.
