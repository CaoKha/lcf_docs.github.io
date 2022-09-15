# LCF-CLUB

### Description:
LCF app is now connected as `CLUB`. In `DEV` or `PROD` environment you will need a `FOOTCLUBS_CONTEXTE` cookie in your browser in order to get an access to the next page. This cookie is set automatically after a successfully authentication to [`FOOTCLUB`](https://footclubspp.fff.fr/) app. After passing this guard, there are 3 pages can be accessed freely: 
- [CLUB-ACCUEIL](./club-accueil.md)
- [CLUB-CANDIDATURE](./club-parameters.md)
- [CLUB-DOSSIER](./club-dossier.md)

### Files:
```ignore
club
|__components
|  |__club-accueil
|  |__club-candidature
|  |__club-dossier
|__club-routing.module.ts
|__club.component.ts 
|__club.component.html
|__club.component.scss
|__club.module.ts
```


#### Routing:
`club-routing.module.ts`:
```ts
const routes: Routes = [
  {
    path: '',
    component: ClubComponent,
    children: [
      {
        path: '404',
        component: PageNotFoundComponent,
      },
      {
        path: ':sa_no',
        component: ClubAccueilComponent,
      },
      {
        path: ':sa_no/lcf',
        component: ClubCandidatureComponent,
      },
      {
        path: ':sa_no/lcf/:competition',
        component: ClubDossierComponent,
      },
    ],
  },
];
```
### Functionality:
`club.component.ts`:

```ts
ngOnInit(): void {
    // redirect to page of the current season
    if (this.router.url === '/club')
      this.router.navigate([String(this.storageService.getSelectedSaisonSaNo),'lcf'],
                            {relativeTo: this.activatedRoute});
}
```

this component is just a parent component who contains 3 others child components:

- [header](./shared/header.md)
- [sidenav](./shared/sidenav.md)
- [club-accueil](./club/club-accueil.md) or [club-candidature](./club/club-candidature.md) or [club-dossier](./club/club-dossier.md) 
 
