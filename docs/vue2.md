# Conventions Vue 2 — stack legacy en class components

> Conventions des frontends Vue 2.7 écrits en composants de classe. Elles décrivent l'existant : on suit le style
> du code en place, on ne migre pas un fichier vers la Composition API ou `<script setup>` au passage.

## Stack
- **Vue 2.7** + **TypeScript** + **Vite**
- **vue-property-decorator / vue-class-component** : composants en classes
- **Vuex 3** + **vuex-module-decorators** (modules dynamiques), `vuex-persist` pour l'état persisté (auth)
- **vue-router 3**, **vue-i18n 8** (locales `fr` et `en`), **Vuetify 2**
- **axios** : une instance unique dans `src/dependencies/http` (token, refresh, erreurs)
- Mocks HTTP avec `axios-mock-adapter`, tests **Vitest** quand le repo en a

## Structure `src/` (par module fonctionnel)
```
src/
├── dependencies/              # plomberie : http, store, router, i18n, vuetify, filters, directives, mixins, modules
├── modules/{module}/
│   ├── index.ts               # classe {Module}Module implements IVueModule : Locales, Routes, mocks
│   ├── components/            # composants .vue du module (pages comprises)
│   ├── configs/               # {domaine}.config.ts : objet `api` des routes backend
│   ├── locales/{fr,en}.json
│   ├── models/I{Type}.d.ts    # un type par fichier, interface préfixée I
│   ├── router/{module}.routes.ts
│   ├── services/{domaine}.service.ts
│   └── store/{domaine}.store.ts
└── shared/                    # core (BaseVue, IVueModule...), helpers, models, composants transverses
```
- Un nouveau module s'enregistre dans `src/dependencies/modules/index.ts` (le module de la page d'accueil reste
  le dernier).

## Composants
- `<script lang="ts">` + `@Component` + `export default class {Nom} extends Vue` (ou `BaseVue`, ou
  `mixins(...)`) ; un composant par fichier, nom de fichier = nom de classe en PascalCase.
- Props via `@Prop`, événements via `@Emit` ou `this.$emit`, watchers via `@Watch` ; état local en champs de
  classe, valeurs dérivées en getters.
- Les données serveur se lisent dans le store (`get clients() { return clientsModule.clients; }`), le chargement
  se déclenche dans `created()` par une action du store.
- Composants transverses dans `shared/components`, préfixés par le préfixe du projet.
- Styles en `<style scoped lang="scss">`, `::v-deep` pour cibler Vuetify.

## Store (vuex-module-decorators)
```ts
@Module({ name: "ArticlesModule", namespaced: true, stateFactory: true, dynamic: true, store, preserveState: false })
class ArticlesModule extends VuexModule {
  articles: IArticle[] = [];

  @Mutation
  setArticles(articles: IArticle[]) {
    this.articles = articles;
  }

  @Action
  public async getArticles() {
    stateModule.setLoading(true);
    try {
      this.setArticles(await getArticles());
    } catch (err) {
      errorHelper.catchErreur(err);
    }
    stateModule.setLoading(false);
  }
}

export const articlesModule = getModule(ArticlesModule);
```
- Un store par écran ou use case (`{domaine}.store.ts`, `{domaine}Detail.store.ts`...), exporté via `getModule`.
- Le loading global passe par `stateModule`, les erreurs par `errorHelper`.

## Appels API
- Routes dans `configs/{domaine}.config.ts` (`export const api = { read: "/Articles/Read", detail(id) {...} }`),
  jamais d'URL en dur dans un service ou un composant.
- Services = fonctions `async` exportées qui appellent l'instance axios de `@/dependencies/http` et renvoient
  `response.data` typé ; jamais d'axios importé directement depuis `axios` hors de `dependencies/http`.
- Les composants n'appellent pas les services : ils passent par les actions du store.

## Routes et i18n
- Routes déclarées par module (`RouteConfig[]`), composants chargés via `lazyView(...)`, `meta` pour le droit,
  le titre et le breadcrumb (clés i18n).
- Aucun texte en dur : `$t("{module}.xxx")` dans le template, `i18n.t(...)` dans le script ; toute clé ajoutée en
  `fr` l'est aussi en `en`.

## TypeScript
- Pas de nouveau `any` : `unknown` si le type est vraiment indéterminé.
- `async/await` plutôt que `.then()/.catch()` dans le code ajouté.
- Imports via l'alias `@/` ; `@krosoft/core` par sous-chemin (`@krosoft/core/helpers`, `@krosoft/core/types`).
