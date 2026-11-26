# Conventions .NET Web API — stack Krosoft.Extensions.WebApi

> Complément de `dotnet.md` pour les projets d'API ASP.NET Core.

## Stack
- **ASP.NET Core Minimal APIs** (pattern `IEndpoint` de `Krosoft.Extensions.WebApi`)

## Organisation en vertical slices
```
src/{Projet}.Api/Features/{Feature}/
├── {Feature}Endpoint.cs
├── {Ressource}Profile.cs                    # AutoMapper commun à plusieurs slices, si besoin
└── {Operation}/
    ├── {Feature}{Operation}Command.cs       # ou Query
    ├── {Feature}{Operation}CommandHandler.cs
    ├── {Feature}{Operation}CommandValidator.cs
    ├── {Feature}{Operation}Profile.cs       # AutoMapper, si besoin
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

## Nommage des contrats d'API (DTO)
- **Entrée HTTP** : binder directement la Command/Query (`[FromBody] CommandeCreateCommand command`,
  `[AsParameters] CommandesQuery query`) ; les ids de route s'injectent via `command with { ArticleId = articleId }`
  (identifiant nommé `{Entity}Id`, comme dans les Commands).
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

## Tests
- Classe de base `{Projet}BaseApiTest` (sur `BaseApiTest<TStartup, TKrosoftContext>` de
  `Krosoft.Extensions.Testing`) pour les tests fonctionnels.
- Package : `Krosoft.Extensions.Testing.WebApi`.
