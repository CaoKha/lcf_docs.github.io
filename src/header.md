# HEADER

The header of the LCF application.

## Description:

This header always appears on top of the application. There is a dropdown box on the right for choosing seasons.

## Files:

```ignore
header.component.html
header.component.scss
header.component.ts
```

## Functionalities:
### General:

```ts
ngOnInit(): void {
  this.currentUser = this.storageService.user;
  this.updateLogoClubUrl();
  this.subs.sink = this.subscribeDossiersFromSaNo(this.spawnSaNoObservable());
  this.subs.sink = this.subscribeParametersFromSaNo(this.spawnSaNoObservable());
}
```

Upon initialization, this header will spawn 2 observables that listen to value changes of `season`.
When a selected `season` changes, it invokes a call to backend and retrieves `dossiers` and `parameters` within the `season`.
Also if connected `user` has a `cl_cod`, we update the club logo on the header.

### Specific:

```ts
private spawnSaNoObservable() {
  return this.storageService.getSelectedSaisonSaNo$.pipe(
    tap((sa_no: number) => {
      this.selectedSaisonSaNo = sa_no;
    }),
    distinctUntilChanged()
  );
}
```

Spawn an observable that listen to value changes of `season`.
Upon receive a new one which is different from the old one (`distincUntilChanged()`), assign the new value to variable `selectedSaisonSaNo`

```ts
private subscribeDossiersFromSaNo(saNo$: Observable<number>) {
  return saNo$
    .pipe(
      switchMap((sa_no: number) => {
        // in case connected as 'club'
        if (
          this.storageService.connectId === 'club' &&
          this.currentUser.cl_cod
        ) {
          return this.dossierService.requestDossiers(
            sa_no,
            this.currentUser.cl_cod
          );
        }
        // case connected as 'dcn', do nothing
        else return of();
      })
    )
    .subscribe({
      next: (dossier: any) => {
        console.log('storage.dossiers: ', dossier);
        this.storageService.dossiers = dossier;
      },
      error: (err: any) => {
        console.error(err);
      },
    });
}
```

Suscribe to the `season` observable, upon a `season` changes, invoke a call to backend, retrieve and set the new `dossiers` of that `season` to the `storage`.

```ts
private subscribeParametersFromSaNo(saNo$: Observable<number>) {
  return saNo$
    .pipe(
      switchMap((sa_no: number) => {
        return this.parametersService.requestParameters(sa_no);
      })
    )
    .subscribe({
      next: (params: any) => {
        console.log('storage.parameters: ', params.results);
        this.storageService.parameters = params.results;
      },
      error: (err: any) => {
        console.error(err);
      },
    });
}
```

Suscribe to the `season` observable, upon a `season` changes, invoke a call to backend, retrieve and set the new `parameters` of that `season` to the `storage`.

```ts
saisonChanged($event: any) {
  if ($event?.target?.value) {
    const sa_no = parseInt($event.target.value);
    this.storageService.selectedSaisonSaNo = sa_no;
    this.changeRouteToSelectedSaisonSaNo(sa_no);
  }
}
```

Upon selecting another `season` in dropdown box, correcting the `season` string in url by using `regex` pattern matching.

```ts
private changeRouteToSelectedSaisonSaNo(sa_no: number): void {
  const newUrl = this.router.url.replace(/\d+/, String(sa_no));
  this.router.navigate([newUrl]);
}
```

Do `regex` pattern matching, replace the `season` string then redirect to that new path.

### HTML:

```html
<header>
  <div class="partG logo cursor-pointer" [routerLink]="['/']">
    <div>
      <div class="filet"></div>
      <div class="projetclub">
        <div class="log"></div>
        <div>Projet <span>Club</span></div>
      </div>
    </div>
  </div>
  <div class="partD" *ngIf="currentUser">
    <div class="saison" *ngIf="storageService.showSaisonBox$ | async">
      <label>Saison</label>
      <select (change)="saisonChanged($event)" [(ngModel)]="selectedSaisonSaNo">
        <option
          *ngFor="let saison of currentUser.saisons"
          [value]="saison?.sa_no"
          [selected]="saison?.sa_no === selectedSaisonSaNo"
        >
          {{saison?.cs_lib}}
        </option>
      </select>
    </div>
    <div class="user">
      <img
        class="logo-club"
        [src]="logoUrl"
        (error)="logoClubErrorHandler($event)"
        alt="Logo Club"
      />
      <div>
        <span class="name">{{currentUser.nom}}</span>
        <span class="club"
          >{{currentUser.cl_cod}} - {{currentUser.club_nom}}</span
        >
      </div>
    </div>
  </div>
</header>

<!-- The header component always appears on top of every page
  (it won't be destroyed when navigating from page to page (page refresh does not count!)) so does the toast.
  Therefore, do not use this toast anywhere else,
  if does, a dubplication of toast will be created on top of each other! -->
<p-toast styleClass="custom-toast" position="bottom-right"></p-toast>
```