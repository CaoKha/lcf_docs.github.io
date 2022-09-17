# LCF-DCN

## Description:

LCF app is now connected as `DCN`. In `DEV` or `PROD` environment you will need a `PORTAILBLEU` cookie in your browser
in order to get an access to the next page. This cookie is set automatically after a successfully authentication to [`PORTAIL_BLEU`](https://portailbleuq.fff.fr/) app. After passing this guard, we will be redirect to [DCN-ACCUEIL](./dcn/dcn-accueil.md). There are also 2 other pages can be accessed freely from here:

- [DCN-PARAMETERS](./dcn/dcn-parameters.md)
- [DCN-DOSSIER](./dcn//dcn-dossier.md)

## Files:

```ignore
dcn
|__components
|  |__dcn-accueil
|  |__dcn-dossier
|  |__dcn-parameters
|__dcn-routing.module.ts
|__dcn.component.ts
|__dcn.component.html
|__dcn.component.scss
|__dcn.module.ts
```

## Routing:

`dcn-routing.module.ts`:

```ts
const routes: Routes = [
  {
    path: "",
    component: DcnComponent,
    children: [
      {
        path: "404",
        component: PageNotFoundComponent,
      },
      {
        path: ":sa_no",
        redirectTo: ":sa_no/lcf",
        pathMatch: "full",
      },
      {
        path: ":sa_no/lcf",
        component: DcnAccueilComponent,
      },
      {
        path: ":sa_no/lcf/params",
        component: DcnParametersComponent,
      },
      {
        path: ":sa_no/lcf/:cl_cod",
        redirectTo: "404",
      },
      {
        path: ":sa_no/lcf/:cl_cod/:competition",
        component: DcnDossierComponent,
      },
    ],
  },
];
```

## Functionalities:

### General:

`dcn.component.ts`:

```ts
ngOnInit(): void {
  // redirect to page of the current season
  if (this.router.url === '/dcn')
    this.router.navigate([String(this.storageService.selectedSaisonSaNo),'lcf'],
                          {relativeTo: this.activatedRoute});
}
```

this component is just a parent component who contains 3 others child components:

- [header](./shared/header.md)
- [sidenav](./shared/sidenav.md)
- [dcn-accueil](./dcn/dcn-accueil.md) or [dcn-parameters](./dcn/dcn-parameters.md) or [dcn-dossier](./dcn/dcn-dossier.md)

### HTML:

```html
<lcf-header></lcf-header>

<lcf-side-nav>
  <router-outlet></router-outlet>
</lcf-side-nav>
```

Plug in [HEADER](./shared/header.md) component and [SIDENAV](./shared/sidenav.md) component.
