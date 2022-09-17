# DCN-PARAMETERS

The dcn parameters page of the LCF application.

## Description:

The dcn parameters page has a table which contains parameters used to calculate a financial aid. 

## Files:

```ignore
dcn-parameters.component.html
dcn-parameters.component.scss
dcn-parameters.component.ts
```

## Functionalities:

### General:

```ts
ngOnInit(): void {
    this.checkSaNoFromUrl();
    this.initTable();
  }
```

This component [checkSaNoFromUrl](#checksanofromurl) and init a `parameters` table within selected `season`.

### Specific:

#### checkSaNoFromUrl

```ts
/**
 * Check `sa_no` in `url` is in range of `user.saisons.sa_no`.
 * If true, set `selectedSaison` in `Store` to `sa_no`.
 * If false, navigate to page 404
 */
private checkSaNoFromUrl(): void {
    const saisonSaNoFromUrl = Number(this.route.snapshot.paramMap.get('sa_no'));
    this.user = this.storageService.user;
    if (this.storageService.checkSaisonInRange(this.user, saisonSaNoFromUrl)) {
        this.storageService.selectedSaisonSaNo = saisonSaNoFromUrl;
    } else {
        this.router.navigate(['/', this.storageService.connectId, '404']);
    }
}
```

This function checks `sa_no` in `url` is in range of `user.saisons.sa_no`.

- If true, set `selectedSaison` in `Store` to `sa_no`
- If false, navigate to page 404

#### initTable

```ts
private initTable() {
  this.cols = [
    {
      data: 'libelle',
      title: 'Compétition',
      editable: false,
      class: '',
      icon: '',
    },
    {
      data: 'date',
      title: 'Date limite candidature',
      editable: true,
      type: 'date',
      class: 'date',
      icon: '',
    },
    {
      data: 'aide_financiere',
      title: 'Aide financière',
      editable: true,
      type: 'number',
      class: 'aide',
      icon: '€',
    },
    {
      data: 'pourcentage_incontournables',
      title: 'Incontournables',
      editable: true,
      type: 'number',
      class: '',
      icon: '%',
    },
    {
      data: 'pourcentage_cumulables_1',
      title: 'Cumulables (> 5 001 pts)',
      editable: true,
      type: 'number',
      class: '',
      icon: '%',
    },
    {
      data: 'pourcentage_cumulables_2',
      title: 'Cumulables (>= 7 500 pts)',
      editable: true,
      type: 'number',
      class: '',
      icon: '%',
    },
  ];
}
```

This function init a `parameters` table structure

#### saveParameters

```ts
saveParameters(parameters: any[]) {
  if (this.verifPourcentage(parameters)) {
    this.convertParamStringToNumber(parameters);
    console.table(parameters);

    this.subs.sink = this.parametersService
      .patchParameters(this.storageService.selectedSaisonSaNo, parameters)
      .subscribe({
        next: (result: any) => {
          this.messageService.add({
            severity: 'success',
            summary: 'Success',
            detail: result.message,
          });
          this.storageService.parameters = parameters;
        },
        error: (err: any) => {
          console.error(err);
          this.messageService.add({
            severity: 'error',
            summary: 'Error',
            detail: err,
          });
        },
      });
  }
}
```

This function save `parameters` to backend.

#### verifPourcentage

```ts
private verifPourcentage(parameters: any[]) {
  let percentageError = [];
  parameters.forEach((param) => {
    if (
      parseInt(param.pourcentage_cumulables_1) +
        parseInt(param.pourcentage_cumulables_2) +
        parseInt(param.pourcentage_incontournables) >
      100
    ) {
      // @ts-ignore
      percentageError.push(param['libelle']);
    }
  });
  if (percentageError.length > 0) {
    this.messageService.add({
      severity: 'error',
      summary: 'Error',
      detail: 'Somme des pourcentages invalide ',
    });
  }
  return percentageError.length <= 0;
}
```

This function applies rule to `parameters` table. If the rule isn't satisfied, we can't [saveParameters](#saveparameters).

#### convertParamStringToNumber

```ts
/**
 * ngModel return a string even if the `type` directive of input = `number`.
 * For that reseason, we have to convert them to number manually
 */
private convertParamStringToNumber(parameters: any[]) {
  parameters.forEach((param: any) => {
    param.aide_financiere = parseInt(param.aide_financiere);
    param.pourcentage_cumulables_1 = parseInt(param.pourcentage_cumulables_1);
    param.pourcentage_cumulables_2 = parseInt(param.pourcentage_cumulables_2);
    param.pourcentage_incontournables = parseInt(
      param.pourcentage_incontournables
    );
  });
}
```

This function converts some strings in `parameters` table to `Integer`.

#### goToAccueil

```ts
goToAccueil() {
  this.router.navigate([
    '/dcn',
    this.storageService.selectedSaisonSaNo,
    'lcf',
  ]);
}
```

These function allows us to go back to [DCN-ACCUEIL](dcn-accueil.md) page.

#### ngOnDestroy

```ts
ngOnDestroy(): void {
    this.subs.unsubscribe();
    // workaround bug: https://github.com/angular/angular/issues/28330
    this.storageService.globalUnsubscribe.next();
    this.storageService.globalUnsubscribe.complete();
}
```

Upon being destroyed, unsubscribe from every dangling observables.

### HTML:

```html
<div
  class="content-pages spe"
  *ngIf="storageService.parameters$ | async; let parameters"
>
  <h1>Paramètres</h1>

  <div class="content margTop30" *ngIf="parameters.length > 0">
    <div class="result">
      <h3><span></span>Paramètres</h3>
    </div>
    <p-table
      [columns]="cols"
      [value]="parameters"
      [responsive]="true"
      styleClass="param-table"
    >
      <ng-template pTemplate="header" let-columns>
        <tr>
          <th *ngFor="let col of columns">{{col.title}}</th>
        </tr>
      </ng-template>
      <ng-template pTemplate="body" let-rowData let-columns="columns">
        <tr id="{{rowData.id}}">
          <td *ngFor="let col of columns">
            <span *ngIf="col.editable">
              <input
                type="{{ col.type }}"
                [(ngModel)]="rowData[col.data]"
                class="{{ col.class }}"
                [min]="0"
              />
              {{ col.icon }}
            </span>
            <span *ngIf="!col.editable"> {{rowData[col.data]}} </span>
          </td>
        </tr>
      </ng-template>
    </p-table>
  </div>
  <div *ngIf="parameters.length === 0">
    <h3>No parameters found</h3>
  </div>

  <div class="buttons">
    <button class="annuler" (click)="goToAccueil()">Annuler</button>
    <button type="submit" class="valider" (click)="saveParameters(parameters)">
      Valider
    </button>
  </div>
</div>
```
