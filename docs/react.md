# Conventions React — stack @krosoft/react

> Conventions des frontends React construits sur `@krosoft/react`.

## Stack
- **React 19** + **TypeScript** (strict) + **Vite** (`@vitejs/plugin-react-swc`), npm, app unique (pas de monorepo)
- **@krosoft/react** + **@krosoft/core** : design system et helpers maison
- **react-router-dom 7** : routes déclarées dans `src/lib/routes.tsx`, pages chargées en `lazy()`
- **@tanstack/react-query 5** : tout l'état serveur ; état global restant en React Context (pas de Redux/Zustand)
- **react-hook-form + zod** : formulaires
- **TailwindCSS** avec le preset `@krosoft/react/tailwind` + composants shadcn fournis par `@krosoft/react/components/ui`
- **i18next / react-i18next** : locales `fr` (fallback) et `en`
- **vite-plugin-pwa** + `PwaUpdater` de `@krosoft/react/pwa`
- Tooling : `@krosoft/tooling-eslint-react`, `@krosoft/tooling-prettier`, `@krosoft/tooling-tsconfig-react`

## Structure `src/` (par couche technique, puis par domaine)
```
src/
├── pages/{domain}/            # pages de route (export default pour lazy)
├── components/{domain}/       # composants métier (dialogs/, tabs/ en sous-dossiers)
├── components/core/           # composants transverses de l'app (layout, auth, states...)
├── hooks/api/                 # un hook React Query par use case
├── hooks/behavior/            # dialogs, workflows, auth, permissions
├── hooks/ui/                  # thème, menu, titre de page...
├── services/http.service.ts   # client HTTP unique (fetch + token + refresh)
├── constants/                 # endpoints.ts (ENDPOINTS), queryKeys.ts (QUERY_KEYS), permissions.ts
├── contexts/  providers/      # *.context.ts, *Provider.tsx
├── schemas/                   # *.schema.ts (zod)
├── helpers/                   # {domain}.helper.ts
├── types/api/                 # un type par fichier, PascalCase
├── lib/                       # i18n.ts, react-query.ts, routes.tsx
└── locales/{fr,en}/           # {domain}.json
```

## Nommage
- Composants : `PascalCase.tsx`, un seul composant par fichier, export nommé en arrow function
  (`export const AddRoleDialog = (...) =>`) ; seules les pages sont en `export default`.
- Composants transverses de l'app préfixés `App` : `AppHeader`, `AppFormField`.
- Props typées via `type {Component}Props`.
- Hooks : `use{Xxx}.ts` (`.tsx` s'ils rendent du JSX).
- Hooks API : `use{Entity}` (query), `use{Entities}Table` (liste paginée), `use{Entity}Manager` (mutations),
  `use{Entities}PickList`.
- Autres fichiers : suffixe en minuscules (`http.service.ts`, `auth.context.ts`, `role.schema.ts`, `article.helper.ts`).
- Imports via l'alias `@/` ; imports Krosoft toujours par sous-chemin (`@krosoft/react/hooks`,
  `@krosoft/core/helpers`), jamais d'import racine.

## Page `/info`
Chaque front expose une route publique `/info` (hors authentification, déclarée avant toute route dynamique
comme `/:id`) qui affiche la version, la date de build et l'environnement ; `/info?format=json` renvoie le JSON
brut. Elle sert à vérifier quel build tourne réellement en prod.

- `vite.config.ts` :
  ```ts
  define: {
    __APP_VERSION__: JSON.stringify(process.env.npm_package_version ?? "0.0.0"),
    __BUILD_DATE__: JSON.stringify(new Date().toISOString()),
    __APP_ENV__: JSON.stringify(mode),
  },
  ```
- `src/vite-env.d.ts` : `declare const __APP_VERSION__: string;` (idem `__BUILD_DATE__`, `__APP_ENV__`).
- `src/pages/Info.tsx` :
  ```tsx
  const info = { version: __APP_VERSION__, buildDate: __BUILD_DATE__, environment: __APP_ENV__ };

  export function Info() {
    const [searchParams] = useSearchParams();
    const isJsonFormat = searchParams.get("format") === "json";

    useEffect(() => {
      if (isJsonFormat) {
        document.body.innerHTML = JSON.stringify(info);
      }
    }, [isJsonFormat]);

    if (isJsonFormat) {
      return null;
    }

    return <pre style={{ fontFamily: "monospace", padding: "1rem" }}>{JSON.stringify(info, null, 2)}</pre>;
  }
  ```

## TypeScript
- `type` plutôt qu'`interface` quand l'héritage n'est pas nécessaire.
- Pas de `any` : `unknown` si le type est vraiment indéterminé.
- `async/await` plutôt que `.then()/.catch()` ; ESM (`import/export`) uniquement.

## Appels API
- Toujours `httpService.request<T>(ENDPOINTS.{domain}.{op}(...))`, jamais de `fetch` direct.
- Clés de cache : `[QUERY_KEYS.{domain}.{key}, id]`.
- Listes paginées : `useTable` (`@krosoft/react/hooks`) + `buildUrl` / `PaginationResult` / `SortOption` (`@krosoft/core`).
- Mutations : `onSuccess` → `showSuccess` (`useNotifications`) + `invalidateQueries`, `onError` → `showError`.

```ts
export function useArticle(articleId?: string) {
  return useQuery({
    queryKey: [QUERY_KEYS.articles.detail, articleId],
    queryFn: () => httpService.request<ArticleDetail>(ENDPOINTS.articles.getById(articleId!)),
    enabled: !!articleId,
  });
}
```

## UI et i18n
- **Toujours** utiliser d'abord ce qu'offre `@krosoft/react` (`AppDialog`, `DataTable`, `AppTabs`, filtres,
  inputs, states, `useConfirmDeleteDialog`...) avant d'écrire un composant ou un hook maison. Un besoin
  générique se propose dans le package plutôt que dans l'app.
- Une série de boutons/actions similaires se déclare dans un tableau de config typé (`DataTableRowAction<T>`,
  `AppAction`, `AppDialogConfig`) rendu par une seule boucle, pas en JSX répété.
- Pas de dossier `src/components/ui` local : les primitives shadcn viennent de `@krosoft/react/components/ui`.
- Aucun texte en dur : clés i18n namespacées par domaine (`administration.roles.notifications.created.title`) ;
  les configs de dialog/actions prennent des clés, pas des chaînes traduites.
- Toute clé ajoutée en `fr` l'est aussi en `en`.

## Packages @krosoft/*
- `@krosoft/core` : helpers agnostiques (url/`buildUrl`,
  erreurs HTTP, dates, fichiers...) et types (`PaginationResult`, `SortOption`, `SelectOption`).
- `@krosoft/react` : primitives shadcn, composants core
  (dialogs, table, filters, inputs, tabs, navbar, layouts, cards, charts), hooks (`useTable`,
  `useNotifications`, `useDialog`...), preset Tailwind, tokens, i18n, PWA. Stories Storybook dans `stories/`.
- `@krosoft/tooling-*` : configs ESLint, Prettier, tsconfig partagées.

Pour comprendre un package npm (Krosoft ou autre), chercher d'abord ses sources si elles sont disponibles (repo
cloné en local, repo public), avant de lire le code compilé dans `node_modules`.

Ne jamais utiliser `npm link`, `yarn link` ni de dépendance `file:` pour tester un `@krosoft/*` : attendre la
publication puis `npm install @krosoft/<pkg>@latest`. Si après la mise à jour le comportement ne correspond pas
au code de `node_modules`, purger `node_modules/.vite` et redémarrer le dev server.
