# LCF-ACCUEIL

## Description:

LCF homepage only appears in `development` environment.
You can either connect to `LCF` as a `CLUB` or `DCN` by clicking on the corresponding route.
It is currently a blank page with 2 clickable links:

- [CLUB](./lcf-club.md)
- [DCN](./lcf-dcn.md)

## Files:

```ignore
lcf-routing.module.ts
lcf.component.ts
lcf.component.html
lcf.component.scss
lcf.module.ts
```

## Routing:

`lcf-routing.module.ts`:

```ts
const routes: Routes = [
  {
    path: "",
    component: LcfComponent,
  },
  {
    path: "dcn",
    loadChildren: () => import(`./dcn/dcn.module`).then((m) => m.DcnModule),
    resolve: { dcn: DcnResolver },
  },
  {
    path: "club",
    loadChildren: () => import("./club/club.module").then((m) => m.ClubModule),
    resolve: { club: ClubResolver },
  },
];
```

The DCN module and CLUB module are lazy loaded when we try to access to [DCN](./lcf-dcn.md) or [CLUB](./lcf-club.md) page.
Note, we are implementing redirection mechanism so we set `useHash` to `true`.

`app-routing.module.ts`:

```ts
@NgModule({
  imports: [
    RouterModule.forRoot(routes, {useHash: true}),
  ],
  exports: [RouterModule],
})
```

Sometimes, the server does not support `HTML5 pushstate` required by `PathLocationStrategy` of `Angular`.
In order to make redirection mechanism work, it is easier to use `HashLocationStrategy`.
Each time we refresh the page, the server will redirect to the current path found in the URL rather than redirect us to the homepage.<br>
With this feature added, user can go directly to the dossier page by typing the dossier path in the URL. This feature is useful when user try to bookmark a dossier page, the next time they open that bookmark, `LCF` app will go directly to the dossier without passing through the homepage.
However, note that there is a [bug](https://github.com/angular/angular/issues/28330) which duplicates a child component while we are using this redirection mechanism.

## Functionalities:

### General:

```ts
ngOnInit(): void {
  this.storageService.user = {};
  this.storageService.cl_cod = 0;
  this.storageService.terrains = [];
  this.storageService.dossiers = [];
  this.storageService.selectedDossier = {};
  this.storageService.selectedSaisonSaNo = 0;
}
```

The page is rendered by the component `lcf.component.ts` and its html `lcf.component.html`.
On initialization, we just reset all internal global states to their initial values.
All the internal global states are store at `storage.service.ts` inside folder `/services`.
This functionality is only used in `development` environment.
