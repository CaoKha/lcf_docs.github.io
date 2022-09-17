# CLUB-DOSSIER

The club dossier page of the LCF application. It is a child component of [DOSSIER-TEMPLATE](../shared/dossier-template.md) component.

## Description:

The club dossier displays a `dossierForm` for user to fill. The `dossierForm` will later be pass to the [DCN-DOSSIER](../dcn/dcn-dossier.md) side for decision.

## Files:

```ignore
club-dossier.component.html
club-dossier.component.scss
club-dossier.component.ts
```

## Functionalities:

### General:

```ts
override ngOnInit(): void {
  this.checkSaNoFromUrl();
  this.getCompetIdFromUrl();
  this.storageService.showSaisonBox$.next(false); // hide saisonBox
  this.updateTerrainsRattachesArray();

  this.dossier = this.storageService.selectedDossier;
  if (this.isEmpty(this.dossier)) {
    this.backupDossier();
  }
  this.setupDossierContent();
  this.spawnClubStatusDossierHandler();
  this.dateLimiteControl();
}
```

Overriding [ngOnInit](../shared/dossier-template.md#general) of [DOSSIER-TEMPLATE](../shared/dossier-template.md). The definition of some functions can be found here if they aren't included in [DOSSIER-TEMPLATE](../shared/dossier-template.md) page.

### Specific:

#### disableDossierFormExceptRefusFile

```ts
private disableDossierFormExceptRefusFile() {
  this.disableDossierForm();
  this.enableRefusFileForm();
}
```

This function disables the `dossierForm` except the `fileForm` field which has an eligibilite of `refus piece`.

#### enableRefusFileForm

```ts
// WIP - generic, if file exist only in incontounables field can do differently
private enableRefusFileForm() {
  for (let formField of UtilsService.fieldArray) {
    let criteresField = this.dossierForm.get(formField) as FormGroup;
    Object.keys(criteresField.controls).forEach((id: string) => {
      if (
        criteresField.get([id, 'type'])?.value === 'PJ' &&
        criteresField.get([id, 'eligibilite'])?.value === 4
      ) {
        criteresField.get(id)?.enable();
      }
    });
  }
}
```

This function enables every `fileForm` field which has an eligibilite of `refus piece`.

#### spawnClubStatusDossierHandler

```ts
private spawnClubStatusDossierHandler() {
  this.subs.sink = this.storageService.selectedDossier$
    .pipe(distinctUntilChanged())
    .subscribe({
      next: (dossier: Dossier) => {
        switch (dossier.id_statuts_dossier_fk) {
          case DossierStatusEnum.REFUS_PIECE:
            this.disableDossierFormExceptRefusFile();
            break;
          case DossierStatusEnum.CLOS:
            this.disableDossierForm();
            break;
          case DossierStatusEnum.A_EVALUER:
            this.disableDossierForm();
            break;
        }
      },
      error: (err: any) => {
        console.error(err);
      },
    });
}
```

This function spawns a `dossier` status handler. Depend on the status of the `dossier`, we disable or enable the corresponding fields.

#### dateLimiteControl

```ts
private dateLimiteControl() {
  this.subs.sink = this.storageService.selectedDossier$
    .pipe(distinctUntilChanged())
    .subscribe({
      next: (dossier: Dossier) => {
        let today = new Date();
        if (dossier.date_limite && dossier.date_limite < today) {
          switch (dossier.id_statuts_dossier_fk) {
            case DossierStatusEnum.NON_DEBUTE:
              this.disableDossierForm();
              this.spawnOverdueToastr();
              this.overdue = true;
              break;
            case DossierStatusEnum.EN_COURS:
              this.disableDossierForm();
              this.spawnOverdueToastr();
              this.overdue = true;
              break;
            case DossierStatusEnum.A_EVALUER:
              this.disableDossierForm();
              this.spawnOverdueToastr();
              this.overdue = true;
              break;
            case DossierStatusEnum.CLOS:
              this.disableDossierForm();
              this.spawnOverdueToastr();
              this.overdue = true;
              break;
            case DossierStatusEnum.REFUS_PIECE:
              this.spawnOverdueToastr();
              this.overdue = true;
              break;
            default:
              this.overdue = false;
              break;
          }
        }
      },
      error: (err: any) => {
        console.error(err);
      },
    });
}
```

This function checks if the `dossier` is overdue or not. If so, disable `dossierForm` and provoke a toastr.

#### spawnOverdueToastr

```ts
private spawnOverdueToastr() {
  this.messageService.add({
    severity: 'error',
    summary: 'Dossier Overdue',
    detail: 'This dossier is overdue!',
  });
}
```

This function spawns the `overdue` toastr.

#### buttonDisplayConditions

```ts
buttonDisplayConditions(dossier: Dossier) {
  // display when not overdue and not 3: dossier clos, and not 4: dossier a evaluer
  return !this.overdue && ![3,4].includes(Number(dossier.id_statuts_dossier_fk));
}
```

This function applies rule for displaying all the buttons at the end of the `dossierForm`.

### HTML:

```html
<div
  class="content-pages"
  *ngIf="storageService.selectedDossier$ | async; let dossier"
>
  <div class="entete grid" [ngClass]="compet_id_icon">
    <div class="mr-5">
      <img class="icon-logo" [src]="dossier.icon_url" alt="icon-competition" />
    </div>
    <div>
      <h2>Licence « Club Fédéral »</h2>
      <h1 *ngIf="user">{{user.cl_cod}} - {{user.club_nom}}</h1>
      <h4 [ngClass]="dossier_status_color">
        {{dossier.status_dossier_libelle | titlecase}}
      </h4>
    </div>
  </div>

  <div class="content">
    <h3><span></span>INFORMATIONS GÉNÉRALES</h3>
    <div class="info">
      <div>
        <span class="bold">Équipe</span>
        <span class="blue">So Romorantin 1</span>
      </div>
      <div>
        <span class="bold">Ligue d’appartenance</span>
        <span class="blue">2100 - Ligue du Centre Val-de-Loire</span>
      </div>
    </div>
    <div
      class="form-content"
      [formGroup]="$any(dossierForm.get('info_general'))"
    >
      <h5>Référent club</h5>
      <div class="ref">
        <div class="ref-item">
          <label for="name">Nom*</label>
          <input
            type="text"
            id="name"
            placeholder="Saisissez votre nom"
            formControlName="nom"
          />
        </div>
        <div class="ref-item">
          <label for="post-name">Prénom*</label>
          <input
            type="text"
            id="post-name"
            placeholder="Saisissez votre prénom"
            formControlName="prenom"
          />
        </div>
        <div class="email ref-item">
          <label for="email">Email*</label>
          <input
            type="text"
            id="email"
            placeholder="Saisissez votre email"
            formControlName="email"
          />
        </div>
        <div class="phone ref-item">
          <label for="phone">Téléphone*</label>
          <input
            type="text"
            id="phone"
            placeholder="Saisissez votre téléphone"
            formControlName="tel"
          />
        </div>
      </div>

      <h5>Autorisations</h5>
      <label class="check">
        <span>
          Le club souhaite obtenir la Licence « Club Fédéral » et confirme ainsi
          son adhésion au dispositif, que l’ensemble des documents sont exacts,
          qu’il autorise la FFF à examiner lesdits documents et à procéder, si
          besoin, à une visite dans les locaux du club.
        </span>
        <input id="autorite" type="checkbox" formControlName="autorisation" />
        <span for="autorite" class="checkmark"></span>
      </label>
    </div>
  </div>

  <div
    *ngFor="let critere_big_title of ['CRITÈRES INCONTOURNABLES','CRITÈRES CUMULABLES']; let i = index"
  >
    <div class="content margTop30">
      <div class="rub">
        <h3>
          <span></span>{{critere_big_title}}<em class="pts"> (5000 POINTS)</em>
        </h3>
        <div>
          <span class="line"></span>
          <span
            class="statut"
            customStyle
            [evaluation]="i === 0 ? (eligibleOfIncon$ | async) : 2 "
          >
            {{ i === 0 ? (eligibleOfIncon$ | async | customString:
            storageService.connectId) : (sumOfCumulable$ | async) + ' points' }}
          </span>
        </div>
      </div>
      <div class="content-dossier">
        <div>
          <div
            *ngFor="let critere_small_title of critereTitleArray[i]; let j = index"
          >
            <div class="grid">
              <h5 class="col-9" [ngClass]="{nomarg: j === 0}">
                {{critere_small_title}}
              </h5>
              <span
                *ngIf="j === 0"
                class="responsiveHide eval col-3 text-center"
                >Évaluation</span
              >
            </div>
            <div *ngIf="critereBigArray">
              <div
                *ngFor="let critere of critereBigArray[i][j]"
                [ngSwitch]="critere.code"
              >
                <div>
                  <lcf-file
                    *ngSwitchCase="'PJ'"
                    (uploadFileCarrier)="uploadFile($event, critereNameArray[i][j], critere.id)"
                    (downloadFileCarrier)="downloadFile($event)"
                    [fileForm]="$any(dossierForm.get(critereNameArray[i][j] + '.' + critere.id.toString()))"
                    [overdue]="overdue"
                  >
                  </lcf-file>
                  <lcf-terrain
                    *ngSwitchCase="'TERRAIN'"
                    [terrainForm]="$any(dossierForm.get(critereNameArray[i][j]+ '.' + critere.id.toString())) "
                    [terrains]="terrains"
                  >
                  </lcf-terrain>
                  <lcf-radio
                    *ngSwitchCase="'RB'"
                    [radioForm]="$any(dossierForm.get(critereNameArray[i][j]+ '.' + critere.id.toString()))"
                  >
                  </lcf-radio>
                  <lcf-wbs
                    *ngSwitchCase="'WBS'"
                    [wbsForm]="$any(dossierForm.get(critereNameArray[i][j]+ '.' + critere.id.toString()))"
                  ></lcf-wbs>
                  <lcf-radio-three
                    *ngSwitchCase="'RB3'"
                    [radioForm]="$any(dossierForm.get(critereNameArray[i][j]+ '.' + critere.id.toString()))"
                  >
                  </lcf-radio-three>
                </div>
              </div>
              <hr
                class="part"
                *ngIf="critere_small_title !== critereTitleArray[i][critereTitleArray[i].length - 1]"
              />
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>

  <div class="content margTop30">
    <div class="rub">
      <span class="arrow"
        ><a href="#"><img src="assets/img/arrow2-down.png" alt="Fermer" /></a
      ></span>
      <h3><span></span>SIMULATION *</h3>
      <div class="spe">
        <span class="line"></span>
      </div>
    </div>
    <div class="content-dossier desk">
      <div class="contG">
        <h5 class="reg nomarg">Critères Incontournables</h5>
        <h5 class="reg margTop2Rem">Critères Cumulables</h5>
      </div>

      <div class="contD crit nobd">
        <span
          class="statut reg nomarg"
          customStyle
          [evaluation]="eligibleOfIncon$ | async"
        >
          {{eligibleOfIncon$ | async | customString: storageService.connectId }}
          - {{(eligibleOfIncon$ | async) === 2 ? 5000 : 0}} point</span
        >
        <span class="statut reg green"
          >{{ sumOfCumulable$ | async}} points</span
        >
      </div>
    </div>

    <hr class="part" />
    <div class="content-dossier desk">
      <div class="contG">
        <h5 class="margTop2Rem">Éligibilité</h5>
        <h5 class="margTop2Rem">Aide financière</h5>
        <em class="ment"
          >* La simulation ne présente pas de caractère officiel et reste
          indicative, dans l’attente de décision définitive du BELFA.</em
        >
      </div>

      <div class="contD crit2 nobd">
        <span
          class="statut reg"
          customStyle
          [evaluation]="eligibleOfIncon$ | async"
        >
          {{eligibleOfIncon$ | async | customString: storageService.connectId }}
          - {{totalPoints$ | async}} points</span
        >
        <span class="statut big" *ngIf="aideFinanciere$ | async; let aideFinan"
          >{{aideFinan.pourcentage}}% - {{aideFinan.totalAide}}€</span
        >
      </div>
    </div>
  </div>

  <div class="buttons" *ngIf="buttonDisplayConditions(dossier)">
    <button class="valider" (click)="showForm()">ShowForm</button>
    <button class="valider" (click)="isFormValid()">isFormValid</button>
    <button
      class="valider"
      *ngIf="dossier.id_statuts_dossier_fk !== 4"
      (click)="saveDossier()"
    >
      Enregistrer
    </button>
    <button class="valider" (click)="validerDossier()">Valider</button>
  </div>
</div>
```
