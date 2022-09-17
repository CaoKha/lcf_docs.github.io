# DCN-ACCUEIL

The dcn homepage of the LCF application.

## Description:

The dcn homepage consists of a filter system and a table which displays all dossiers within a selected season.

## Files:

```ignore
dcn-accueil.component.html
dcn-accueil.component.scss
dcn-accueil.component.ts
```

## Functionalities:

### General:

```ts
ngOnInit(): void {
  this.checkSaNoFromUrl();
  const filtres: FiltresList = this.storageService.filtres;
  this.competitions = filtres.competitions;
  this.incontournables = filtres.incontournables;
  this.statuts = filtres.statuts;
  this.belfas = filtres.belfas;
  this.subs.sink = this.storageService.selectedSaisonSaNo$
    .pipe(
      takeUntil(this.storageService.globalUnsubscribe),
      switchMap((sa_no: number) => {
        return this.dossierService.getFilteredDossiers(
          sa_no,
          this.filtreForm
        );
      })
    )
    .subscribe((result: any) => {
      this.dcnTable = result;
    });
}
```

WIP

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

#### onRecherche

```ts
onRecherche() {
  this.subs.sink = this.dossierService
    .getFilteredDossiers(
      this.storageService.selectedSaisonSaNo,
      this.filtreForm
    )
    .subscribe((result: any) => {
      this.dcnTable = result;
      console.log('onRecherche, dcnTable: ', result);
    });
}
```

This function request a filtered list of club dossiers within the selected `season` from backend and store them in `dcntable` variable.

#### onReset

```ts
onReset() {
  this.filtreForm.reset();
}
```

This function resets the filter.

#### goToDossier

```ts
goToDossier(compet_id: number, cl_cod: number) {
  // reset menuItem highlight
  this.storageService.firstMenuItemActivated = false;
  this.storageService.secondMenuItemActivated = false;

  this.storageService.cl_cod = cl_cod;
  this.subs.sink = this.dossierService
    .requestDossier(
      this.storageService.selectedSaisonSaNo,
      cl_cod,
      compet_id
    )
    .pipe(
      combineLatestWith(
        this.dossierService.requestTerrainsAttachesAuxClubs(cl_cod)
      )
    )
    .subscribe(([dossier, terrains]: [any, any]) => {
      console.log('storage.terrains:', terrains);
      console.log('storage.dossier: ',dossier);
      this.storageService.selectedDossier = dossier;
      this.storageService.terrains = terrains;
      this.router.navigate([
        `/dcn/${
          this.storageService.selectedSaisonSaNo
        }/lcf/${cl_cod}/${UtilsService.getCompetitionByCode(compet_id)}`,
      ]);
    });
}
```

Upon click on a club `dossier` among list of club `dossiers`, we will be redirect to the [DCN-DOSSIER](dcn-dossier.md) page. Before directing to the page, we request from backend all `terrains` attached to that selected club along with all information about that `dossier`.

#### isRefus

```ts
isRefus(belfa: string) {
  if (belfa.indexOf('Refus') !== -1) {
    return true;
  } else return false;
}
```

This function returns `true` if it found `Refus` string in the `belfa` variable. This will later be used as a condition to or not to display motif popup when hovering above the `belfa status`.

#### exportExcel, saveAsExcelFile

```ts
exportExcel() {
  const modifiedTable = this.renameKeyInDcnTable(this.dcnTable);

  import('xlsx').then((xlsx) => {
    const worksheet = xlsx.utils.json_to_sheet(modifiedTable);
    const workbook = { Sheets: { data: worksheet }, SheetNames: ['data'] };
    const excelBuffer: any = xlsx.write(workbook, {
      bookType: 'xlsx',
      type: 'array',
    });
    this.saveAsExcelFile(excelBuffer, 'resultats');
  });
}

saveAsExcelFile(buffer: any, fileName: string): void {
  let EXCEL_TYPE =
    'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet;charset=UTF-8';
  let EXCEL_EXTENSION = '.xlsx';
  const data: Blob = new Blob([buffer], {
    type: EXCEL_TYPE,
  });
  saveAs(
    data,
    fileName +
      '-' +
      new Date().toDateString().replace(/ /g, '') +
      EXCEL_EXTENSION
  );
}
```

These function are used for export the `dcnTable` as an `Excel` format file.

#### renameKeyInDcnTable

```ts
/**
 * return a `COPY` of `dcnTable` with modified headers
 */
renameKeyInDcnTable(dcnTable: any[]) {
  return dcnTable
    .slice()
    .map(({ compet_id, ...theRest }) => theRest)
    .map((el) => ({
      Saison: el.sa_no,
      Compétition: el.competition,
      Club: el.club,
      Incontournables: el.incontournables,
      Cumulables: el.cumulables,
      Statut: el.statut,
      'Décision BELFA': el.decision_belfa,
    }));
}
```

This function return a copy of `dcnTable` with modified readable string headers.

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
<div class="content-pages spe">
  <h1>Recherche / Liste</h1>
  <div class="content">
    <div class="form-content">
      <div class="ref filtre columns" [formGroup]="filtreForm">
        <div class="ref-item">
          <label>Compétition</label>
          <p-multiSelect
            formControlName="competitions"
            [options]="competitions"
            defaultLabel="Sélectionner la compétition"
            display="chip"
            optionLabel="libelle"
            optionValue="code"
          >
          </p-multiSelect>
        </div>
        <div class="ref-item">
          <label>Club</label>
          <input
            class="club-name"
            type="text"
            formControlName="club"
            placeholder="Saisissez le nom du club"
          />
        </div>
        <div class="ref-item">
          <label>Incontournables</label>
          <p-multiSelect
            formControlName="incontournables"
            [options]="incontournables"
            defaultLabel="Sélectionner l'éligibilité"
            display="chip"
            optionLabel="libelle"
            optionValue="id"
          >
          </p-multiSelect>
        </div>
        <div class="ref-item cumul">
          <label>Cumulables</label>
          <p-dropdown
            [options]="cumulables"
            formControlName="cumulables"
            optionValue="code"
            optionLabel="libelle"
            placeholder="Sélectionner le nombre de points"
          >
          </p-dropdown>
        </div>
        <div class="ref-item">
          <label>Statut</label>
          <p-multiSelect
            formControlName="statuts"
            [options]="statuts"
            defaultLabel="Sélectionner le statut du dossier"
            display="chip"
            optionLabel="libelle"
            optionValue="id"
          >
          </p-multiSelect>
        </div>
        <div class="ref-item">
          <label>Décision BELFA</label>
          <p-multiSelect
            formControlName="belfas"
            [options]="belfas"
            defaultLabel="Sélectionner la décision BELFA"
            display="chip"
            optionLabel="libelle"
            optionValue="id"
          >
          </p-multiSelect>
        </div>
      </div>
    </div>
  </div>

  <div class="buttons">
    <button
      class="annuler"
      (click)=" onReset();"
      style="margin-right: 20px; height: 3.6rem"
    >
      <span></span>Réinitialiser
    </button>
    <button class="rechercher" (click)=" onRecherche();">
      <span class="recherche-icon"></span>Rechercher
    </button>
  </div>

  <div class="content margTop30">
    <div class="result">
      <h3><span></span>RÉSULTATS</h3>
      <button class="download" (click)="downloadXLS.click()">
        <span></span>Télécharger
      </button>
    </div>
    <div class="flex" style="justify-content:end; ">
      <button
        type="button"
        pButton
        pRipple
        icon="pi pi-file-excel"
        (click)="exportExcel()"
        #downloadXLS
        class="p-button-success mr-2"
        pTooltip="XLS"
        tooltipPosition="bottom"
        style="display: none"
      ></button>
    </div>
    <p-table
      class="dcn-table"
      [value]="dcnTable"
      [paginator]="true"
      [rows]="10"
      [showCurrentPageReport]="false"
      currentPageReportTemplate="{currentPage}"
      responsiveLayout="scroll"
      [rowsPerPageOptions]="[10,20,30]"
      [showJumpToPageDropdown]="true"
      [showPageLinks]="false"
    >
      <ng-template pTemplate="header">
        <tr style="font-size: 1.4rem;">
          <th pSortableColumn="competition">
            Compétition<p-sortIcon field="competition"></p-sortIcon>
          </th>
          <th pSortableColumn="club">
            Club<p-sortIcon field="club"></p-sortIcon>
          </th>
          <th class="incon-col" pSortableColumn="incontournables">
            Incontournables<p-sortIcon field="incontournables"> </p-sortIcon>
          </th>
          <th class="cumul-col" pSortableColumn="cumulables">
            Cumulables<p-sortIcon field="cumulables"></p-sortIcon>
          </th>
          <th pSortableColumn="statut">
            Statut<p-sortIcon field="statut"></p-sortIcon>
          </th>
          <th class="belfa-col" pSortableColumn="decision_belfa">
            Décision BELFA<p-sortIcon field="decision_belfa"></p-sortIcon>
          </th>
          <th></th>
        </tr>
      </ng-template>
      <ng-template pTemplate="body" let-dossier>
        <tr style="font-size: 1.4rem">
          <td style="color: #01407D">{{dossier.competition ?? '-'}}</td>
          <td style="color: #01407D">
            <span pTooltip="Equipe: nom de l'equipe"
              >{{dossier.club ?? '-'}}</span
            >
          </td>
          <td class="incon-col" customStyle [libelle]="dossier.incontournables">
            {{dossier.incontournables ?? '-'}}
          </td>
          <td class="cumul-col" style="color: #01407D">
            {{dossier.cumulables ?? '-'}}
          </td>
          <td customStyle [libelle]="dossier.statut">
            {{dossier.statut ?? '-'}}
          </td>
          <td class="belfa-col" *ngIf="isRefus(dossier.decision_belfa)">
            <span
              customStyle
              [libelle]="dossier.decision_belfa"
              pTooltip="Motif: {{dossier.motif_dossier}}"
              >{{dossier.decision_belfa ?? '-'}}
            </span>
          </td>
          <td
            class="belfa-col"
            *ngIf="!isRefus(dossier.decision_belfa)"
            customStyle
            [libelle]="dossier.decision_belfa"
          >
            {{dossier.decision_belfa | customString }}
          </td>
          <td class="edit" style="cursor: pointer;">
            <img
              src="../../../../../assets/img/pix-edit.png"
              width="20px"
              height="20px"
              alt="Edition"
              (click)="goToDossier(dossier.compet_id, dossier.cl_cod)"
            />
          </td>
        </tr>
      </ng-template>
      <ng-template pTemplate="paginatorleft" let-state>
        {{state.totalRecords > 0 ? state.first + 1 : state.first}} à
        {{(state.first + state.rows <= state.totalRecords) ? (state.first +
        state.rows) : state.totalRecords}} / {{state.totalRecords}} élément(s)
      </ng-template>
    </p-table>
  </div>
</div>
```
