# Conventions Supabase — stack @krosoft/react-supabase

> Complément de `react.md` pour les frontends React adossés à Supabase.

## Ce qui remplace `react.md`
- Les données se lisent et s'écrivent via le client Supabase (`supabase.from(...)`, `.rpc(...)`,
  `.functions.invoke(...)`, `.storage`) dans les hooks `hooks/api/`, à la place de
  `httpService.request<T>(ENDPOINTS...)`. Pas de `fetch` direct vers Supabase.
- Noms de tables centralisés dans `constants/tableNames.ts` (`TABLE_NAMES`), à la place de `constants/endpoints.ts`.
- Listes paginées sur une table : `useSupabaseTable`, à la place de `useTable` + `buildUrl`.
- `services/http.service.ts` n'existe que si l'app appelle aussi une API HTTP maison ; il prend alors le token
  d'accès de la session Supabase (pas de file de refresh, Supabase rafraîchit seul).

## Structure
```
src/
├── integrations/supabase/
│   ├── client.ts             # client unique createClient<Database>, lu depuis VITE_SUPABASE_URL / VITE_SUPABASE_PUBLISHABLE_KEY
│   └── types.ts              # types de la base générés (Database) : jamais édités à la main
├── providers/AuthProvider.tsx # enveloppe l'AuthProvider de @krosoft/react-supabase/providers
├── constants/tableNames.ts
└── types/supabase/           # SupabaseFilter, SupabaseKey...
supabase/
├── config.toml
├── migrations/               # {yyyyMMddHHmmss}_{description_snake_case}.sql
└── functions/{nom-kebab}/    # edge functions ; code commun dans functions/_shared/
```

## Accès aux données
- Un seul client : toujours importer `supabase` depuis `@/integrations/supabase/client`, jamais de second `createClient`.
- Chaque appel teste l'erreur et la relance pour React Query : `const { data, error } = await ...; if (error) throw error;`.
- Le mapping des colonnes `snake_case` vers les types applicatifs `camelCase` se fait dans le hook (ou un helper),
  pas dans les composants.
- Clés de cache, mutations et notifications : mêmes règles que `react.md` (`QUERY_KEYS`, `invalidateQueries`,
  `showSuccess` / `showError`).
- Pas de clé `service_role` côté front : seule la clé publishable est exposée ; la sécurité repose sur les
  politiques RLS.

## Base et edge functions
- Toute modification de schéma passe par une migration versionnée dans `supabase/migrations/`, jamais par le
  dashboard seul ; une migration publiée ne se modifie plus, on en ajoute une nouvelle.
- Toute nouvelle table a RLS activé avec ses politiques dans la même migration.
- Après une migration, régénérer `integrations/supabase/types.ts`.
- Un traitement qui demande un secret ou un appel externe protégé va dans une edge function, appelée via
  `supabase.functions.invoke`.

## Auth
- Session, connexion et déconnexion via `@krosoft/react-supabase` (`AuthProvider`, hooks associés) : ne pas
  réimplémenter la gestion de session.
