# CLUB-CANDIDATURE

The club candidature page of the LCF application.

## Description:

The club candidature displays all the competitions within the selected season of a club. Note, there is also a display rule to follow which is: There can only be one or less competition of type `NATIONAL`.

The priority is `NATIONAL 1` > `NATIONAL 2` > `NATIONAL 3`.

## Files:

```ignore
club-candidature.component.html
club-candidature.component.scss
club-candidature.component.ts
```

## Functionalities:

### General:

```ts
ngOnInit() {
  this.checkSaNoFromUrl();
  const selectedSaNo = this.storageService.selectedSaisonSaNo;
  if (this.storageService.dossiers?.[0].sa_no !== selectedSaNo) {
    this.backupDossiers(selectedSaNo);
  }
  this.spawnDossiersObservableThenFilterDossiers();
  this.title.setTitle(
    UtilsService.createTitle('Votre dossier de candidature')
  );
}
```

When we open this page, it will check the `season` string in the `url` is indeed in range of `seasons` defined in `user.saisons` ([checkSaNoFromUrl](#checksanofromurl)). Then it stores the `season` value in `storage` . Else, redirect to [PAGE-NOT-FOUND](../shared/page-not-found.md) if the `season` is out of range.

Secondly, if the `dossiers` season is different from the `season` found in the `url`. Request another `dossiers` from backend which have the matching `season`.

Thirdly, apply the display rule for every `dossiers` received from the backend.

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

#### backupDossiers

```ts
/**
 * Request `dossiers` from backend
 */
private backupDossiers(sa_no: number) {
  if (this.storageService.user.cl_cod) {
    this.subs.sink = this.dossierService
      .requestDossiers(sa_no, this.storageService.user.cl_cod)
      .pipe(takeUntil(this.storageService.globalUnsubscribe))
      .subscribe((dossiers: any) => {
        this.storageService.dossiers = dossiers;
      });
  }
}
```

This function request a `dossiers` within selected `season` from backend in demand.

#### downloadFile

```ts
async downloadFile() {
  const file = {
    nature: 'reglement',
    categorie: 'documentation',
  };
  this.subs.sink = this.dossierService
    .requestDownloadFile(file.categorie, file.nature)
    .subscribe({
      next: (result: any) => {
        UtilsService.open_file(result);
      },
      error: (err: any) => {
        console.error(err);
      },
    });
}
```

This function is used for download the file from `AzureStorage` attached on the page. We marked it `async` to indicate that it uses a `fetch` API (which is asynchronous) inside the [open_file]() function.

#### ngOnDestroy

```ts
ngOnDestroy(): void {
    this.sub.unsubscribe();
    // workaround bug: https://github.com/angular/angular/issues/28330
    this.storageService.globalUnsubscribe.next();
    this.storageService.globalUnsubscribe.complete();
}
```

Upon being destroyed, unsubscribe `selectedSaisonSaNo$` stream and every dangling observables.

### HTML:

```html
<div class="content-pages">
  <div class="entete">
    <h2>Licence « Club Fédéral »</h2>
    <h1>{{storageService.cl_cod}} - {{storageService.user.club_nom}}</h1>
  </div>

  <div class="content">
    <h3><span></span>Processus</h3>
    <p>
      Les clubs amateurs des championnats de D1 FEMININE, NATIONAL, NATIONAL 2,
      NATIONAL 3 et D1 FUTSAL (ci-après dénommés « candidats ») peuvent postuler
      à la délivrance de la Licence « Club Fédéral » en faisant acte de
      candidature. La délivrance de la Licence « Club Fédéral » est décidée en
      cours de saison par le Bureau Exécutif de la LFA. Les clubs à statut
      professionnel, évoluant en D1 FEMININE, NATIONAL, NATIONAL 2, NATIONAL 3
      et D1 FUTSAL, y compris les équipes réserves, ne peuvent candidater au
      dispositif.
      <br /><br />
      La délivrance de la Licence est réalisée en Saison N sur la base des
      critères du niveau de compétition de la même saison N pour les clubs visés
      ci-dessus. La délivrance de la Licence Club Fédéral déclenche le versement
      d’une aide financière dont le montant est défini par le Comité Exécutif de
      la FFF (COMEX), sur proposition du Bureau Exécutif de la LFA (BELFA).
      <br /><br />
      La participation d'un club à l’un des cinq championnats susvisés n’est pas
      conditionnée par la délivrance de la Licence « Club Fédéral ». Il en est
      de même pour les modalités d’accession et relégation dans ces
      championnats.
      <br /><br />
      La procédure à suivre pour la délivrance de la Licence Club Fédéral, ainsi
      que les critères devant être remplis par le club, sont définis dans le
      règlement disponible
      <a download="Règlement" (click)="downloadFile()" class="cursor-pointer"
        >ici</a
      >.
    </p>
  </div>

  <div class="content margTop30">
    <h3><span></span>Dossiers de candidatures</h3>
    <p class="int">
      <strong>Sélectionner le dossier auquel vous souhaitez accéder :</strong>
    </p>

    <div
      class="content-dossiers"
      *ngIf="filteredDossiers$ | async; let dossiers"
    >
      <div class="dossiers" *ngFor="let dossier of dossiers">
        <a href="#" (click)="goToDossier($event, dossier)">
          <div
            class="blc1"
            [ngClass]="{'national': dossier.id_ref_competitions_fk ? dossier.id_ref_competitions_fk <= 3 : false,
          'feminine':  dossier.id_ref_competitions_fk === 4,
          'futsal': dossier.id_ref_competitions_fk === 5}"
          >
            <span class="bold black"
              >{{dossier.competitions_libelle?.toUpperCase()}}</span
            >
            <!--            Todo Equipe-->
            <span>*Equipe</span>
            <span
              class="bold red"
              [ngClass]="{'red': dossier.id_statuts_dossier_fk === 1,
            'yellow': dossier.id_statuts_dossier_fk === 2,
            'green': dossier.id_statuts_dossier_fk === 3}"
              >{{dossier.status_dossier_libelle | titlecase}}</span
            >
            <span class="bold"
              >Date limite : {{dossier.date_limite | date: 'dd/MM/yyyy'}}</span
            >
          </div>
        </a>
        <div class="blc2">
          <div class="lab">
            <span>Incontournables :</span>
            <span>Cumulables :</span>
            <span class="bold">Éligibilité :</span>
          </div>
          <div class="rep">
            <span customStyle [evaluation]="dossier.id_ref_statuts_critere_fk">
              <!--              TODO add FFF if status = A évaluer -->
              <!--              TODO, mettre des tirets quand y'a rien -->
              {{dossier.elligibilite_incontournables_libelle}}</span
            >
            <span
              *ngIf="dossier.nb_points_criteres_cumulables"
              [ngClass]="{'blue': dossier.nb_points_criteres_cumulables ? dossier.nb_points_criteres_cumulables <=0 : false,
            'green': dossier.nb_points_criteres_cumulables ? dossier.nb_points_criteres_cumulables > 0 : false}"
            >
              {{ dossier.nb_points_criteres_cumulables <= 0 ? '-' :
              dossier.nb_points_criteres_cumulables + ' points' }}</span
            >
            <span
              [ngClass]="{'blue': dossier.id_eligibilite_fk === 3,
            'red': dossier.id_eligibilite_fk === 4,
            'green': dossier.id_eligibilite_fk === 2}"
            >
              <strong
                >{{dossier.id_eligibilite_fk ? (dossier.eligibilite_libelle |
                titlecase) : '-'}}</strong
              >{{dossier.id_eligibilite_fk != 2 ? '' : ' - ' +
              dossier.nb_points_total + ' points'}}
            </span>
          </div>
        </div>
        <div class="blc3">
          <div class="lab">
            <span class="bold">Décision BELFA :</span>
          </div>
          <div class="rep">
            <a href="#" (click)="openModal($event, dossier)">
              <span
                class="alert"
                [ngClass]="{'red': dossier.id_ref_type_decision_belfa_fk === 2,
            'green': dossier.id_ref_type_decision_belfa_fk === 1,
            'blue': !dossier.id_ref_type_decision_belfa_fk}"
              >
                <strong *ngIf="dossier.id_ref_type_decision_belfa_fk"
                  >{{dossier.type_decision_belfa_libelle | titlecase}}
                </strong>
                ({{!dossier.id_ref_type_decision_belfa_fk ? '-' :
                dossier.belfa_date | date: 'dd/MM/yyyy'}} )
              </span>
            </a>
          </div>
        </div>
      </div>
    </div>
  </div>
</div>

<div id="ModalGen" class="modal" [ngClass]="{'show': showModal}">
  <div class="modal-background" (click)="closeModal($event)"></div>
  <div class="modal-content">
    <span class="close"
      ><a href="#" (click)="closeModal($event)">&nbsp;</a></span
    >
    <div
      class="content"
      *ngIf="storageService.selectedDossier$ | async; let selectedDossier"
    >
      <h3><span></span>DÉCISION BELFA</h3>
      <div class="ent">
        <div>
          <span class="bold margBot10">Date décision</span>
          <span class="blue"
            >{{selectedDossier.belfa_date | date: 'dd/MM/yyyy'}}</span
          >
        </div>
        <div>
          <span class="bold margBot10">Décision</span>
          <span class="blue"
            >{{selectedDossier.type_decision_belfa_libelle}}</span
          >
        </div>
      </div>
      <div class="margTop10">
        <span class="bold margBot10">Motif</span>
        <span class="blue"
          >{{selectedDossier.motifs_decision_belfa_libelle}}</span
        >
      </div>
      <div class="margTop10">
        <span class="bold margBot10">Commentaire</span>
        <span class="blue">{{selectedDossier.belfa_commentaire}}</span>
      </div>
    </div>
  </div>
</div>
```
