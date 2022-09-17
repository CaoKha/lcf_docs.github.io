# DCN-DOSSIER

The dcn dossier page of the LCF application. It is a child component of [DOSSIER-TEMPLATE](../shared/dossier-template.md) component.

## Description:

The dcn dossier displays a `dossierForm` for user to fill. This [DCN](../dcn/dcn-dossier.md) dossier version could be considered as an `administration` of the [CLUB](../club/club-dossier.md) version. This means everything a club can do, so does the `DCN` and more.

## Files:

```ignore
dcn-dossier.component.html
dcn-dossier.component.scss
dcn-dossier.component.ts
```

## Functionalities:

### General:

```ts
override ngOnInit(): void {
  this.checkSaNoFromUrl();
  this.getClCodFromUrl();
  this.getCompetIdFromUrl();
  this.storageService.showSaisonBox$.next(false); // hide saisonBox
  if (!this.isEmpty(this.storageService.terrains)) {
    this.updateTerrainsRattachesArray();
  } else {
    this.backupTerrains();
  }
  this.dossier = this.storageService.selectedDossier;
  if (this.isEmpty(this.dossier)) {
    this.backupDossier();
  }
  this.setupDossierContent();
  this.spawnDcnStatusDossierHandler();
}
```

Overriding [ngOnInit](../shared/dossier-template.md#general) of [DOSSIER-TEMPLATE](../shared/dossier-template.md). The definition of some functions can be found here if they aren't included in [DOSSIER-TEMPLATE](../shared/dossier-template.md) page.

### Specific:

#### getClCodFromUrl

```ts
private getClCodFromUrl() {
    const cl_cod_url = Number(this.route.snapshot.paramMap.get('cl_cod'));
    if (this.storageService.cl_cod !== cl_cod_url) {
      this.storageService.cl_cod = cl_cod_url;
      console.log('cl_cod has been set with cl_cod in url!');
    }
  }
```

This function retrieves the `cl_cod` string in url at put it into `storage`.

#### backupTerrains

```ts
/**
 * Request `terrains` from backend then do `updateTerrainsRattachesArray` function
 */
private backupTerrains() {
  this.subs.sink = this.dossierService
    .requestTerrainsAttachesAuxClubs(this.storageService.cl_cod)
    .subscribe({
      next: (terrains: any) => {
        this.storageService.terrains = terrains;
        this.updateTerrainsRattachesArray();
        console.log('backupTerrainsAttaches: ', terrains);
      },
      error: (err: any) => {
        console.error(err);
      },
    });
}
```

This function requests `terrains` from backend then do the [updateTerrainsRattachesArray](../shared/dossier-template.md#updateterrainsrattachesarray) function

#### isACumulCriterePointNull

```ts
private isACumulCriterePointNull(dossierForm: FormGroup) {
  let foundNull: boolean = false;
  for (let formField of UtilsService.cumulArray) {
    let criteresField = dossierForm.get(formField) as FormGroup;

    if (
      Object.keys(criteresField.controls).find(
        (id: string) =>
          criteresField.get(id + '.nb_points_calcule')?.value === null
      )
    ) {
      foundNull = true;
      break;
    }
  }
  return foundNull;
}
```

This function verifies if there exists a critere cumulable that has a `null` points.

#### validerDossier

```ts
override validerDossier(): void {
  let isValid = this.isDossierFormValid(this.dossierForm);
  let eligibiliteValid = this.isEligibiliteInconValid();
  if (this.dossier.id && isValid && eligibiliteValid) {
    this.sendDossierToBackend(
      true,
      this.storageService.connectId,
      'Validation réussi'
    );
  } else {
    if (!this.dossier.id) {
      console.error('dossier.id is undefined or null');
      console.info('dossier info:', this.dossier);
    } else if (!isValid) {
      console.error('The dossierForm is invalid!');
      console.info('dossierForm info:', this.dossierForm);
    } else if (!eligibiliteValid) {
      console.error(
        'Le statut des critères incontournables doit être différent à "à évaluer"!'
      );
      this.messageService.add({
        severity: 'error',
        summary: 'Erreur',
        detail:
          'Le statut des critères incontournables doit être différent à "à évaluer"',
      });
    }
  }
}
```

This function overrides the original [validerDossier](../shared/dossier-template.md#validerdossier). This newly version will also add another stricter verification (if `Criteres Incontournables` eligibilite is valid) and then sending `dossier` to backend.

#### validerDecision

```ts
validerDecision() {
  this.updateDecisionValidators();
  let eligibiliteValid = this.isEligibiliteInconValid();
  let isDossierValid = this.isDossierFormValid(this.dossierForm);
  if (this.decisionForm.valid && isDossierValid && eligibiliteValid) {
    this.sendDecisionToBackend('Decision envoyé');
    this.closeBothConfirmAndDecisionModal();
  } else {
    if (!this.decisionForm.valid)
      this.messageService.add({
        severity: 'error',
        summary: 'Erreur',
        detail: 'Decision est invalide',
      });
    else if (!eligibiliteValid) {
      this.messageService.add({
        severity: 'error',
        summary: 'Erreur',
        detail:
          'Le statut des critères incontournables doit être différent à "à évaluer"',
      });
    }
    this.closeConfirmOpenModal();
  }
}
```

This function checks if the `dossierForm` is valid, it also checks `Criteres Incontournables` eligibilite and then sending the `decisionForm` to backend. This function will later be used in [makeDecision](#makedecision).

#### isEligibiliteInconValid

```ts
private isEligibiliteInconValid() {
  if (this.eligibleOfIncon$.value === EligibiliteEnum.A_EVALUER) return false;
  else return true;
}
```

This function checks if `Criteres Incontournables` eligibilite is valid.

#### updateDecisionValidators

```ts
private updateDecisionValidators() {
  // if not refus, reset commentaire and motif
  if (this.decisionForm.value.decision !== 2) {
    this.decisionForm.patchValue({
      commentaire: '',
      motif_decision: null,
    });
    this.decisionForm.get('motif_decision')?.clearValidators();
    this.decisionForm.get('commentaire')?.clearValidators();
  } else {
    this.decisionForm
      .get('motif_decision')
      ?.setValidators(Validators.required);
    this.decisionForm.get('commentaire')?.setValidators(Validators.required);
  }
  this.decisionForm.get('motif_decision')?.updateValueAndValidity();
  this.decisionForm.get('commentaire')?.updateValueAndValidity();
}
```

This function applies rules for updating `Validators` for `decisionForm`.

#### sendDecisionToBackend

```ts
private sendDecisionToBackend(success_message: string) {
  this.subs.sink = this.dossierService
    .sendDecision(
      Number(this.dossier.id),
      this.decisionForm
    )
    .pipe(
      switchMap((send_decision_response: any) => {
        console.log(
          'dossier is sent, send_dossier_response:',
          send_decision_response
        );
        return this.dossierService.requestDossier(
          this.storageService.selectedSaisonSaNo,
          this.storageService.cl_cod,
          this.storageService.competId
        );
      })
    )
    .subscribe({
      next: (dossier: Dossier) => {
        this.storageService.selectedDossier = dossier;
        console.log('storage.dossier: ', dossier);
        this.messageService.add({
          severity: 'success',
          summary: 'Succès',
          detail: success_message,
        });
      },
      error: (err: any) => {
        console.error(err);
      },
    });
}
```

This function sends `decisionForm` to backend and then updating the `dossier` in `storage`.

#### resetMotifDecision

```ts
resetMotifDecision() {
  this.decisionForm.patchValue({ motif_decision: null, commentaire: '' });
}
```

This function resets the `motif` field inside the `decisionForm`. It is used when user changing their `decision` in `decisionForm`.

#### spawnDcnStatusDossierHandler

```ts
private spawnDcnStatusDossierHandler() {
  this.subs.sink = this.storageService.selectedDossier$
    .pipe(distinctUntilChanged())
    .subscribe({
      next: (dossier: Dossier) => {
        if (dossier.id_statuts_dossier_fk === DossierStatusEnum.CLOS) {
          this.disableDossierForm();
        }
      },
      error: (err: any) => {
        console.error(err);
      },
    });
}
```

This function spawns a `dcn` version of dossier status handler. We only disable the `dossierForm` if and only if the dossier status is `CLOS`.

#### reOpenDossier

```ts
reOpenDossier() {
  this.subs.sink = this.dossierService
    .requestReOpenDossier(Number(this.dossier.id))
    .pipe(
      switchMap((reopen_response: any) => {
        console.log('reopen_response:', reopen_response);
        return this.dossierService.requestDossier(
          this.storageService.selectedSaisonSaNo,
          this.storageService.cl_cod,
          this.storageService.competId
        );
      })
    )
    .subscribe({
      next: (dossier: Dossier) => {
        this.storageService.selectedDossier = dossier;
        this.enableDossierForm();
        console.log('storage.dossier:', dossier);
        this.messageService.add({
          severity: 'success',
          summary: 'Succès',
          detail: 'Dossier est réouvert',
        });
      },
      error: (err: any) => {
        console.error(err);
      },
    });
}
```

This function allows `dcn` user to unlock a `CLOS` dossier. After unlocking, the dossier status will becomes `EN COURS`.

#### makeDecision

```ts
/**
 * open decision modal only when `eligibiliteOfIncon` is diffenrent from `a_evaluer` and `refus_piece`
 */
makeDecision() {
  if (
    [1, 4].includes(this.eligibleOfIncon$.value) ||
    this.isACumulCriterePointNull(this.dossierForm)
  ) {
    this.validerDossier();
  } else {
    this.showModal = true;
  }
}
```

This function is called after user click on `decision` button. If `dossier` status is different of `a evaluer` and `refus piece` and there is no `null points` critere cumulable, we open the `decisionForm` modal. Else, execute [validerDossier](#validerdossier) function.

### HTML:

```html
<div
  class="content-pages"
  *ngIf="storageService.selectedDossier$ | async; let dossier"
>
  <form action="traitement.php">
    <div class="entete grid" [ngClass]="compet_id_icon">
      <div class="mr-5">
        <img
          class="icon-logo"
          [src]="dossier.icon_url"
          alt="icon-competition"
        />
      </div>
      <div>
        <h2>Licence « Club Fédéral »</h2>
        <h1 *ngIf="user">{{storageService.cl_cod}} - {{dossier.cl_nom}}</h1>
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
            Le club souhaite obtenir la Licence « Club Fédéral » et confirme
            ainsi son adhésion au dispositif, que l’ensemble des documents sont
            exacts, qu’il autorise la FFF à examiner lesdits documents et à
            procéder, si besoin, à une visite dans les locaux du club.
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
            <span></span>{{critere_big_title}}<em class="pts">
              (5000 POINTS)</em
            >
          </h3>
          <div>
            <span class="line"></span>
            <span
              class="statut"
              customStyle
              [evaluation]="i === 0 ? (eligibleOfIncon$ | async) : 2 "
            >
              {{ i === 0 ? (eligibleOfIncon$ | async | customString:
              storageService.connectId) : (sumOfCumulable$ | async) + ' points'
              }}
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
            {{eligibleOfIncon$ | async | customString: storageService.connectId
            }} - {{(eligibleOfIncon$ | async) === 2 ? 5000 : 0}} point</span
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
            {{eligibleOfIncon$ | async | customString: storageService.connectId
            }} - {{totalPoints$ | async}} points</span
          >

          <span
            class="statut big"
            *ngIf="aideFinanciere$ | async; let aideFinan"
            >{{aideFinan.pourcentage}}% - {{aideFinan.totalAide}}€</span
          >
        </div>
      </div>
    </div>

    <div
      class="buttons"
      *ngIf="dossier.id_statuts_dossier_fk !== 3; else reopenButton"
    >
      <button class="valider" (click)="showForm()">ShowForm</button>
      <button class="valider" (click)="isFormValid()">isFormValid</button>
      <button
        class="valider"
        *ngIf="dossier.id_statuts_dossier_fk !== 4"
        (click)="saveDossier()"
      >
        Enregistrer
      </button>
      <button class="valider" (click)="makeDecision()">Decision</button>
    </div>
    <ng-template #reopenButton>
      <div class="buttons">
        <button class="valider" (click)="reOpenDossier()">Réouverture</button>
      </div>
    </ng-template>
  </form>
</div>

<div id="ModalGen" class="modal" [ngClass]="{'show': showModal}">
  <div class="modal-background" (click)="closeModal($event)"></div>
  <div class="modal-content">
    <span class="close"
      ><a href="#" (click)="closeModal($event)">&nbsp;</a></span
    >
    <div
      class="content"
      *ngIf="storageService.selectedDossier$| async; let dossier"
      [formGroup]="decisionForm"
    >
      <h3><span></span>DÉCISION BELFA</h3>
      <div class="ent">
        <div>
          <span class="bold margBot10">Date décision</span>
          <input
            class="blue"
            type="date"
            formControlName="date_decision"
            [ngClass]="{'input-error': addDateDecisionControl && !decisionForm.controls.date_decision.valid}"
            value="{{dossier.belfa_date | date: 'dd/MM/yyyy'}}"
          />
        </div>
        <div>
          <span class="bold margBot10">Décision</span>

          <select
            class="blue"
            formControlName="decision"
            (change)="resetMotifDecision()"
          >
            <option value="null" disabled selected>
              Sélectionner décision
            </option>
            <option
              *ngFor="let decision of storageService.filtres.decisions"
              [ngValue]="decision.id"
            >
              {{decision.libelle}}
            </option>
          </select>
        </div>
      </div>
      <div class="margTop10" *ngIf="decisionForm.value.decision === 2">
        <span class="bold margBot10">Motif</span>
        <select class="blue" formControlName="motif_decision">
          <option value="Sélectionner decision" disabled selected>
            Sélectionner motif
          </option>
          <option
            *ngFor="let motif of storageService.filtres.motifs"
            [ngValue]="motif.id"
          >
            {{motif.libelle}}
          </option>
        </select>
      </div>
      <div class="margTop10" *ngIf="decisionForm.value.decision === 2">
        <span class="bold margBot10">Commentaire</span>
        <textarea class="blue" formControlName="commentaire">
{{dossier.belfa_commentaire}}</textarea
        >
      </div>
    </div>
    <div class="buttons">
      <button class="valider" (click)="openConfirmCloseModal()">Valider</button>
    </div>
  </div>
</div>

<div id="ModalGen2" class="modal" [ngClass]="{'show': showConfirm}">
  <div class="modal-background" (click)="closeConfirmOpenModal()"></div>
  <div class="modal-content">
    <span class="close" style="cursor: pointer;"
      ><a (click)="closeConfirmOpenModal()">&nbsp;</a></span
    >
    <div class="content">
      <h3><span></span>Confirmation</h3>
      <div>Etes-vous sur avec votre decision?</div>
      <div class="buttons">
        <button class="annuler" (click)="closeConfirmOpenModal()">
          Annuler
        </button>
        <button class="valider" (click)="validerDecision()">Valider</button>
      </div>
    </div>
  </div>
</div>
```
