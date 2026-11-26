# Conventions .NET — stack Krosoft.Extensions

> Socle commun aux repos .NET.

## Stack
- **.NET 10+** pour les nouveaux projets
- **MediatR** — CQRS (commands / queries / handlers), **FluentValidation**
- **Entity Framework Core** — PostgreSQL
- **AutoMapper** via `Krosoft.Extensions.Mapping`
- **Krosoft.Extensions.*** — CQRS, Data, Mapping, Testing : toujours les préférer aux solutions tierces
- AutoMapper et MediatR sont figés sur leur dernière version open source (les suivantes sont sous licence
  commerciale) : jamais de montée de version.
- Pour comprendre un package NuGet (Krosoft ou autre), chercher d'abord ses sources si elles sont disponibles
  (repo cloné en local, repo public), avant de décompiler la DLL ou de fouiller le cache NuGet.

## Style C#
- Une seule classe par fichier.
- Commentaire au-dessus d'une méthode ou d'une classe : `/// <summary>` (doc XML), jamais `//` ; les `//` sont
  réservés au corps du code.
- Records et primary constructors quand c'est pertinent.
- Pattern matching et expressions `switch` plutôt que des `if/else` en cascade.
- Namespaces file-scoped (`namespace X;`), namespace = chemin de dossier.
- Nommage : `PascalCase` pour types, méthodes, propriétés et namespaces ; `camelCase` pour les variables
  locales ; `_camelCase` pour les champs privés ; interfaces préfixées `I`.
- Suffixe explicite selon le rôle : `*Handler`, `*Validator`, `*Service`, `*Configuration`, `*Converter`,
  `*Extensions`.

## Commands et Queries
```csharp
internal record ArticleDeleteCommand(Guid ArticleId) : BaseCommand;
internal record ArticlesDisponiblesQuery(Guid CategorieId) : BaseQuery<IEnumerable<ArticleDto>>;
```

## Données
- `IReadRepository<T>` / `IWriteRepository<T>` génériques de `Krosoft.Extensions.Data` + `IUnitOfWork`.
  Jamais de `DbContext` injecté dans un handler, jamais de repository concret.
- Configurations EF Core dans `Data/Configurations/{Entity}Configuration.cs` (`EntityTypeConfiguration<T>`).

## Mapping (AutoMapper)
- Un `Profile` par fichier, `internal`. Mapping propre à un slice : `{Feature}{Operation}Profile` dans le dossier
  du slice. Mapping commun à plusieurs slices d'une feature (DTO de ressource partagé) : `{Ressource}Profile` à
  la racine de la feature.
- Mapping toujours explicite : chaque membre cible par `ForMember(...MapFrom(...))`, puis
  `.ForAllOtherMembers(m => m.Ignore())` (`Krosoft.Extensions.Mapping.Extensions`). Aucun mapping implicite par
  convention de nom.

```csharp
internal class ArticleCreateProfile : Profile
{
    public ArticleCreateProfile()
    {
        CreateMap<ArticleCreateCommand, Article>()
            .ForMember(dest => dest.Libelle, o => o.MapFrom(src => src.Libelle))
            .ForMember(dest => dest.Prix, o => o.MapFrom(src => src.Prix))
            .ForAllOtherMembers(m => m.Ignore());
    }
}
```

## DI
- Extension methods par feature / module : `.AddArticles()`, `.Add{Projet}()`.

## Configuration projet
`Directory.Build.props` : `ImplicitUsings`, `Nullable` et `TreatWarningsAsErrors` activés. Un warning se
corrige, il ne se supprime pas.

## Tests
- Projets dans `tests/`, suffixés `.Tests`, répartis en `Unit/` et `Functional/`, socle dans `Core/`.
- Framework **MSTest** (jamais xUnit ni NUnit), assertions **NFluent** (`Check.That(...)`, jamais `Assert.*`),
  mocks **Moq**. Les trois sont déclarés en `<Using Include="...">` dans le `Directory.Build.props` de `tests/`,
  pas réimportés dans les fichiers.
- Classe de base `{Projet}BaseTest` (sur `BaseTest` de `Krosoft.Extensions.Testing`), seed via
  `{Projet}SeedService` enregistré par `AddSeedService<TContext, TSeed>()`, constantes dans
  `{Projet}TestConstants`.
- EF Core **in-memory** via `AddDbContextInMemory<TContext>(true)`, jamais de mock d'EF Core ni de vraie base.
- Nommage : classe `{ComposantTesté}Tests` (jamais `{ComposantTesté}Should`), méthodes en français métier
  (`Create_ArticleSansLibelle_LeveUneException`).
- Packages : `Krosoft.Extensions.Testing`, `Krosoft.Extensions.Testing.Cqrs`.
