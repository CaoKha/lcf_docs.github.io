# DOSSIER-TEMPLATE

The template (parent component) for both [DCN-DOSSIER](../dcn/dcn-dossier.md) and [CLUB-DOSSIER](../club/club-dossier.md).

## Description:

This component stores all core functionalities that will later be used by [DCN-DOSSIER](../dcn/dcn-dossier.md) component and [CLUB-DOSSIER](../club/club-dossier.md) component. These functionalities can be easily overriden if [DCN-DOSSIER](../dcn/dcn-dossier.md) or [CLUB-DOSSIER](../club/club-dossier.md) has different implementations from the original implementation.

## Files:

```ignore
dossier-template.component.ts
```

## Functionalities:

### General:

```ts
ngOnInit(): void {}
```

This is only a template that will later be used/overriden by [DCN-DOSSIER](../dcn/dcn-dossier.md) component and [CLUB-DOSSIER](../club/club-dossier.md) component. There will be no html or scss render by this component.

### Specific:

#### initCriteresFormAndCriteresArrays

```ts
initCriteresFormAndCriteresArrays(criteres: Critere[]) {
  for (let critere of criteres) {
    this.initCriteresForm(this.dossierForm, critere);
    switch (critere.id_ref_thematique_fk) {
      case 1:
        this.incon_struct_org.push(critere);
        break;
      case 2:
        if (critere.id_ref_type_critere_fk === 1)
          this.incon_insta_sport.push(critere);
        else if (critere.id_ref_type_critere_fk === 2)
          this.cumul_insta_sport.push(critere);
        break;
      case 3:
        if (critere.id_ref_type_critere_fk === 1)
          this.incon_trans_finan.push(critere);
        else if (critere.id_ref_type_critere_fk === 2)
          this.cumul_trans_finan.push(critere);
        break;
      case 4:
        this.cumul_struct_equip.push(critere);
        break;
      case 5:
        this.cumul_proj_sport.push(critere);
        break;
    }
  }
  // side effect: store all arrays in bigArray, later be used in html template
  this.critereBigArray = [
    [this.incon_struct_org, this.incon_insta_sport, this.incon_trans_finan],
    [
      this.cumul_insta_sport,
      this.cumul_struct_equip,
      this.cumul_trans_finan,
      this.cumul_proj_sport,
    ],
  ];
}
```

The objective of this function is to initiate the [`criteresForm`](#initcriteresform) of `dossierForm` and build a `critereBigArray` structure that match the dossier form in the UI. The `dossierForm` has the following structure:

- Criteres Incontournables (`id_ref_type_critere_fk === 1`):
  - Structuration et organisation du club
  - Installations Sportives
  - Transparence financière
- Criteres Cumulables (`id_ref_type_critere_fk === 2`):
  - Installations Sportives
  - Structuration des équipes et encadrement sportif
  - Transparence financière
  - Projet Sportif Jeunes

#### updateTerrainsRattachesArray

```ts
/**
   * Remove terrains with `te_nom` == `STADE EN SUPRESSION` and construct
   * `title` field in order to display it in select box
   */
updateTerrainsRattachesArray() {
  this.terrains = [];
  for (let terrain of this.storageService.getTerrains) {
    if (terrain.te_nom !== 'STADE EN SUPPRESSION')
      this.terrains.push({
        nni: terrain.nni,
        classement_eclairage: terrain.classement_eclairage,
        classement_installation: terrain.classement_installation,
        title:
          String(terrain.nni) +
          ' - ' +
          terrain.te_loca +
          ' - ' +
          terrain.te_nom,
      });
  }
}
```

This function is used to build an array that contains all the `filtered` terrains attached to the club. It is used in the select dropdown box in the `html`. The filtered array is a list of terrains that has `te_nom` different from `STADE EN SUPPRESSION`. Also note that, the list of terrains in `storage` has already been sorted by `nni` in the backend so we don't need to sort them in the frontend.

#### initCriteresForm

```ts
private initCriteresForm(dossierForm: FormGroup, critere: Critere) {
  switch (critere.id_ref_type_critere_fk) {
    case 1: // incontournables
      switch (critere.id_ref_thematique_fk) {
        case 1: // struct_org
          this.buildField('incon_struct_org', dossierForm, critere);
          break;
        case 2: // installation sportive
          this.buildField('incon_insta_sport', dossierForm, critere);
          break;
        case 3: // transparence financiere
          this.buildField('incon_trans_finan', dossierForm, critere);
          break;
      }
      break;
    case 2: // cumulables
      switch (critere.id_ref_thematique_fk) {
        // case 1: // struct_org
        //   this.buildField('cumul_struct_org', dossierForm, critere);
        //   break;
        case 2: // installation sportive
          this.buildField('cumul_insta_sport', dossierForm, critere);
          break;
        case 3: // transparence financiere
          this.buildField('cumul_trans_finan', dossierForm, critere);
          break;
        case 4:
          this.buildField('cumul_struct_equip', dossierForm, critere);
          break;
        case 5:
          this.buildField('cumul_proj_sport', dossierForm, critere);
          break;
      }
      break;
  }
}

private buildField(
  field_name: string,
  dossierForm: FormGroup,
  critere: Critere
) {
  switch (critere.code) {
    case 'TERRAIN':
      this.addTerrain(dossierForm, field_name, critere);
      break;
    case 'RB':
      this.addRadio(dossierForm, field_name, critere);
      break;
    case 'RB3':
      this.addRadioThree(dossierForm, field_name, critere);
      break;
    case 'PJ':
      this.addFile(dossierForm, field_name, critere);
      break;
    case 'WBS':
      this.addRadio(dossierForm, field_name, critere);
      break;
  }
}

private addTerrain(
  dossierForm: FormGroup,
  form_field: string,
  critere: Critere
) {
  let fieldGroup = dossierForm.get(form_field) as FormGroup;

  fieldGroup.addControl(
    String(critere.id),
    this.fb.group({
      search: [''],
      rattache: [critere.valeur],
      terrain: [
        critere.nni
          ? {
              classement_eclairage: critere.classement_eclairage,
              classement_installation: critere.classement_installation,
              nni: critere.nni,
            }
          : null,
      ],
      libelle: [critere.libelle],
      id: [critere.id],
      type: [critere.code],
      eligibilite: [critere.eligibilite],
      ref_eligibilite: [critere.eligibilite],
      nb_points_calcule: [critere.nb_points_calcule],
      nb_points_max: [critere.nb_points_max],
      nb_points_min: [critere.nb_points_min],
      code: [critere.id_ref_thematique_fk],
      modification_manuelle: [critere.modification_manuelle],
    })
  );
}

private addRadio(
  dossierForm: FormGroup,
  form_field: string,
  critere: Critere
) {
  let fieldGroup = dossierForm.get(form_field) as FormGroup;
  fieldGroup.addControl(
    String(critere.id),
    this.fb.group({
      valeur: [critere.valeur],
      libelle: [critere.libelle],
      id: [critere.id],
      type: [critere.code],
      eligibilite: [critere.eligibilite],
      ref_eligibilite: [critere.eligibilite],
      nb_points_calcule: [critere.nb_points_calcule],
      nb_points_max: [critere.nb_points_max],
      nb_points_min: [critere.nb_points_min],
      code: [critere.id_ref_thematique_fk],
    })
  );
}

private addRadioThree(
  dossierForm: FormGroup,
  form_field: string,
  critere: Critere
) {
  let fieldGroup = dossierForm.get(form_field) as FormGroup;
  fieldGroup.addControl(
    String(critere.id),
    this.fb.group({
      valeur: [critere.valeur],
      libelle: [critere.libelle],
      id: [critere.id],
      type: [critere.code],
      eligibilite: [critere.eligibilite],
      ref_eligibilite: [critere.eligibilite],
      nb_points_calcule: [critere.nb_points_calcule],
      nb_points_max: [critere.nb_points_max],
      nb_points_min: [critere.nb_points_min],
      radio_label: [critere.radio_label],
      code: [critere.id_ref_thematique_fk],
    })
  );
}

private addFile(
  dossierForm: FormGroup,
  form_field: string,
  critere: Critere
) {
  let fieldGroup = dossierForm.get(form_field) as FormGroup;
  fieldGroup.addControl(
    String(critere.id),
    this.fb.group({
      statut_general: [critere.statut_manager_general],
      file: [
        null,
        !critere.eligibilite || critere.eligibilite === 4
          ? Validators.required
          : null,
      ],
      azure_name: [critere.azure_name],
      file_name: [critere.file_name],
      new_file_name: [null],
      ref_file_name: [critere.file_name],
      libelle: [critere.libelle],
      id: [critere.id],
      type: [critere.code],
      eligibilite: [critere.eligibilite],
      ref_eligibilite: [critere.eligibilite], // previous status stored in database
      motif_pj: [critere.motif_pj],
      ref_motif_pj: [critere.motif_pj], // previous motif stored in database
      code: [critere.id_ref_thematique_fk],
    })
  );
}
```

Above are `critreresForm` dynamic building functions. Depending on the `critere.type` (PJ, TERRAIN, RB, RB3 or WBS), we'll generate the corresponding form control. It will be used together with [initInfoGenerale](#initinfogenerale) to create a `dossierForm`.

#### initInfoGenerale

```ts
private initInfoGenerale() {
  let info_general = this.dossierForm.get('info_general') as FormGroup;
  info_general.patchValue({
    nom: this.dossier.referent_nom,
    prenom: this.dossier.referent_prenom,
    email: this.dossier.referent_mail,
    tel: this.dossier.referent_telephone,
    autorisation: this.dossier.autorisation_adhesion,
  });
}
```

This function initiate the `Information Generale` field of `dossierForm`. It will later be used together with [initCriteresForm](#initcriteresform) to create a `dossierForm`

#### resetAllCritereArrays

```ts
/**
 * In order to avoid the bug where child component rerenders 2 times,
 * we reset all arrays before pushing new criteres to those arrays
 */
resetAllCritereArrays() {
  this.incon_struct_org = [];
  this.incon_insta_sport = [];
  this.incon_trans_finan = [];
  this.cumul_insta_sport = [];
  this.cumul_trans_finan = [];
  this.cumul_struct_equip = [];
  this.cumul_proj_sport = [];
}
```

This function resets all critere arrays. It is needed to overcome a known bug in Angular: <https://github.com/angular/angular/issues/28330>

#### isEmpty

```ts
/**
 * Check if object is an empty array or an empty json
 */
isEmpty(object: any) {
  if (object) return Object.keys(object).length === 0;
  else return true;
}
```

This is just a helper function which helps to detect whether an object is empty or not.

#### backupDossier

```ts
/**
 * Request `dossier` from backend.
 * In case not found, navigate to page `404`
 */
backupDossier() {
  const sa_no = this.storageService.selectedSaisonSaNo;
  const cl_cod = this.storageService.cl_cod;
  const compet_id = this.storageService.competId;
  this.subs.sink = this.dossierService
    .requestDossier(sa_no, cl_cod, compet_id)
    .pipe(takeUntil(this.storageService.globalUnsubscribe))
    .subscribe({
      next: (dossier: Dossier) => {
        console.log('backupDossier: ', dossier);
        this.dossier = dossier;
        this.storageService.selectedDossier = dossier;
      },
      error: (err: any) => {
        console.error(err);
        this.router.navigate(['/', this.storageService.connectId, '404']);
      },
    });
}
```

This function is called if and only if user refresh the dossier page. When the page is refreshed, `LCF` will reset its internal memory which leads to no `dossier` state in `storage`. Therefore, we have to get another `dossier` from backend and reput it into the frontend `storage`.

#### validerDossier

```ts
validerDossier() {
  let isValid = this.isDossierFormValid(this.dossierForm);
  if (this.dossier.id && isValid) {
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
    }
  }
}
```

This function is called after user click on `VALIDER` button on the `dossier` page. It will check if `dossierForm` is valid then send the `dossierForm` to backend.

#### saveDossier

```ts
/**
 * save `dossier` without checking if form is valid
 * still checking if info_general is valid though
 */
saveDossier() {
  let info_general = this.dossierForm.get('info_general') as FormGroup;
  let is_info_general_valid = this.isInfoGeneralValid(info_general);
  if (this.dossier.id && is_info_general_valid) {
    this.sendDossierToBackend(
      false,
      this.storageService.connectId,
      'Enregistrement réussi'
    );
  } else {
    if (!this.dossier.id) {
      console.error('dossier.id is undefined or null');
      console.info('dossier info:', this.dossier);
    } else if (!is_info_general_valid) {
      console.error('The info_general is invalid!');
      console.info(
        'dossierForm info:',
        this.dossierForm.controls['info_general']
      );
    }
  }
}
```

This function is called after user click on `ENREGISTRER` button on the `dossier` page. It will check only if `Information Generale` field is valid then send the `dossierForm` to backend.

#### sendDossierToBackend

```ts
sendDossierToBackend(
  isDossierValid: boolean,
  connectId: string,
  success_message: string
) {
  this.subs.sink = this.dossierService
    .sendDossier(
      Number(this.dossier.id),
      Number(this.dossier.id_statuts_dossier_fk),
      this.dossierForm,
      isDossierValid,
      connectId,
      this.eligibleOfIncon$.value,
      this.sumOfCumulable$.value,
      this.totalPoints$.value,
      this.aideFinanciere$.value
    )
    .pipe(
      switchMap((result: any) => {
        console.log('backendResponse:', result);
        const dossier$ = this.dossierService.requestDossier(
          this.storageService.selectedSaisonSaNo,
          this.storageService.cl_cod,
          this.storageService.competId
        );
        const dossiers$ = this.dossierService.requestDossiers(
          this.storageService.selectedSaisonSaNo,
          this.storageService.cl_cod
        );
        return combineLatest([dossiers$, dossier$]);
      })
    )
    .subscribe(([dossiers, dossier]) => {
      console.log('storage.dossiers:', dossiers);
      console.log('storage.selectedDossier:', dossier);
      this.storageService.selectedDossier = dossier;
      this.storageService.dossiers = dossiers;
      this.messageService.add({
        severity: 'success',
        summary: 'Succès',
        detail: success_message,
      });
    });
}
```

This function is used to send a `dossierForm` with some other important variables (.e.g `points calculation`) to backend. It's also allow us to customize the toastr `success_message` which appears after the `dossierForm` has been successfully sent.
The `isDossierValid` parameter is just there to differentiate situations where user clicked on `ENREGISTRER` button (dossier not valid but it's ok to be sent) or `VALIDER` button.

|                | VALIDER | ENREGISTRER |
| -------------- | ------- | ----------- |
| isDossierValid | true    | false       |

#### isDossierFormValid

```ts
isDossierFormValid(dossierForm: FormGroup): boolean {
  let info_general = dossierForm.get('info_general') as FormGroup;
  let validToken1 = this.isInfoGeneralValid(info_general);
  let validToken2: boolean = true;
  for (let formField of UtilsService.fieldArray) {
    let criteresField = dossierForm.get(formField) as FormGroup;
    validToken2 =
      this.isCritereValid(criteresField, formField) && validToken2;
  }
  return validToken1 && validToken2;
}
```

This function return wheter or not a dossier is `valid`.

#### isInfoGeneralValid

```ts
private isInfoGeneralValid(info_general: FormGroup): boolean {
  let validToken: boolean = true;
  // check validity of nom and prenom

  for (let field of ['nom', 'prenom']) {
    if (
      !info_general.controls[field].valid &&
      info_general.controls[field].enabled
    ) {
      this.messageService.add({
        severity: 'error',
        summary: UtilsService.toReadableString(field) + ' Incomplet',
        detail: 'Veuillez saisir le ' + field + ' du référent du dossier',
        // sticky: true,
      });
      validToken = false;
    }
  }

  // check validity of email
  if (
    !info_general.controls['email'].valid &&
    info_general.controls['email'].enabled
  ) {
    this.messageService.add({
      severity: 'error',
      summary: 'Email Invalide',
      detail: 'Mail du référent invalide',
      // sticky: true,
    });
    validToken = false;
  }

  // check validity of telephone
  if (
    !info_general.controls['tel'].valid &&
    info_general.controls['tel'].enabled
  ) {
    this.messageService.add({
      severity: 'error',
      summary: 'Telephone Invalide',
      detail:
        'Numéro de téléphone invalide. Caractères autorisés: chiffres, +, -, (), espaces',
      // sticky: true,
    });
    validToken = false;
  }

  // check validity of autorisation
  if (
    !info_general.controls['autorisation'].valid &&
    info_general.controls['autorisation'].enabled
  ) {
    this.messageService.add({
      severity: 'error',
      summary: 'Autorisations Incomplet',
      detail:
        "Veuillez accepter les mentions d'autorisation FFF en haut de la page",
      // sticky: true,
    });
    validToken = false;
  }

  return validToken;
}
```

This function returns whether or not the `Information Generale` field is valid. It will later be used in [isDossierFormValid](#isdossierformvalid) function. Note that, a `disabled` field will be considered as `valid` too.

#### isCritereValid

```ts
private isCritereValid(criteresField: FormGroup, formField: string): boolean {
  let validToken = true;
  Object.keys(criteresField.controls).forEach((id: string, index: number) => {
    let position = index + 1;
    switch (criteresField.get([id, 'type'])?.value) {
      case 'PJ':
        if (
          !criteresField.get([id, 'file'])?.valid &&
          criteresField.get([id, 'file'])?.enabled
        ) {
          this.messageService.add({
            severity: 'error',
            summary:
              UtilsService.toReadableString(formField) +
              ': Critère ' +
              position +
              ' Incomplet',
            detail: 'Veuillez insérer une pièce jointe',
            // sticky: true,
          });
          validToken = false;
        }
        break;

      case 'TERRAIN':
        if (
          !criteresField.get([id, 'terrain'])?.valid &&
          criteresField.get([id, 'terrain'])?.enabled
        ) {
          this.messageService.add({
            severity: 'error',
            summary:
              UtilsService.toReadableString(formField) +
              ': Critère ' +
              position +
              ' Incomplet',
            detail:
              'Vous devez sélectionner une installation dans la liste déroulante, ou cliquer sur "Pas d\'installation"',
            // sticky: true,
          });
          validToken = false;
        }
        break;

      case 'RB':
        if (
          !criteresField.get([id, 'valeur'])?.valid &&
          criteresField.get([id, 'valeur'])?.enabled
        ) {
          this.messageService.add({
            severity: 'error',
            summary:
              UtilsService.toReadableString(formField) +
              ': Critère ' +
              position +
              ' Incomplet',
            detail: 'Vous devez donner une réponse (Oui ou Non)',
            // sticky: true,
          });
          validToken = false;
        }
        break;

      case 'RB3':
        if (
          !criteresField.get([id, 'valeur'])?.valid &&
          criteresField.get([id, 'valeur'])?.enabled
        ) {
          this.messageService.add({
            severity: 'error',
            summary:
              UtilsService.toReadableString(formField) +
              ': Critère ' +
              position +
              ' Incomplet',
            detail: 'Vous devez donner une réponse (Oui ou Non)',
            // sticky: true,
          });
          validToken = false;
        }
        break;

      case 'WBS':
        //TODO
        break;
    }
  });
  return validToken;
}
```

This function returns whether or not a `Critere` field is valid. It will later be used in [isDossierFormValid](#isdossierformvalid) function. Note that, a `disabled` field will be considered as `valid` too. If a `critere` is not valid, we provide an error message indicates which and where the invalid event occurs. Putting the `formField` into function [toReadableString]() will result in a more comprehensive critere location message.

#### setCompetIdIcon

```ts
// set up classes with conditions later be used by ngClass in the html template
setCompetIdIcon() {
  this.compet_id_icon = {
    national: Number(this.dossier.id_ref_competitions_fk) >= 1,
    feminin: this.dossier.id_ref_competitions_fk === 4,
    futsal: this.dossier.id_ref_competitions_fk === 5,
  };
}
```

This function sets up a ngClass for competition icon later be used in the html template.

#### setDossierStatusColor

```ts
// set up classes with conditions later be used by ngClass in the html template
setDossierStatusColor() {
  this.dossier_status_color = {
    red: this.dossier.id_statuts_dossier_fk === 1,
    yellow: this.dossier.id_statuts_dossier_fk === 2,
    green: this.dossier.id_statuts_dossier_fk === 3,
  };
}
```

This function sets up a ngClass for dossier statut color later be used in the html template.

#### checkSaNoFromUrl

```ts
/**
 * Check `sa_no` from `url` is in range of `user.saisons.sa_no`.
 * If true, set `selectedSaison` in `Store` to `sa_no`.
 * If false, navigate to page 404
 */
checkSaNoFromUrl(): void {
  const saisonSaNoFromUrl = Number(this.route.snapshot.paramMap.get('sa_no'));
  this.user = this.storageService.user;
  if (this.storageService.checkSaisonInRange(this.user, saisonSaNoFromUrl)) {
    this.storageService.selectedSaisonSaNo = saisonSaNoFromUrl;
  } else {
    this.router.navigate(['/', this.storageService.connectId, '404']);
  }
}
```

This function check `sa_no` from `url` to see if it is in range of `user.saisons.sa_no`.

- If true, set `selectedSaison` in `storage` to `sa_no`
- If false, navigate to page 404

#### getCompetIdFromUrl

```ts
getCompetIdFromUrl() {
  const competition = this.route.snapshot.paramMap.get('competition');
  if (competition) {
    this.storageService.competId = Number(
      UtilsService.getCodeByCompetition(competition)
    );
  }
}
```

This function get the `competition` string in `url`, convert it to a number then store it in `storage`.

#### uploadFile

```ts
uploadFile(file: any, field_name: string, critere_id: number) {
  let fileForm = this.dossierForm.get([
    field_name, // form field which has critere.code == 'PJ' .i.e 'incon_struct_org'
    String(critere_id),
  ]);
  if (file) {
    const reader: FileReader = new FileReader();
    reader.readAsDataURL(file); // read as base64
    reader.onload = () => {
      const { categorie, nature } = UtilsService.getCategorieNatureById(
        fileForm?.value.id
      );
      const fileToUpload = {
        file_name: file.name,
        content: String(reader.result).split(',')[1], // strip the first part before the comma
        content_type: file.type,
        categorie: categorie,
        nature: nature,
      };
      // upload a file with content
      fileForm?.patchValue({
        file: fileToUpload,
        eligibilite: 1, // set status to en attente FFF after uploading a file
        motif_pj: '', // reset motif_pj to ''
      });
    };
  }
  // upload nothing (file == null)
  else {
    fileForm?.patchValue({
      file: null,
      // if file uploading is null, restore eligibilite to the eligibilite value in backend
      eligibilite: fileForm.value.ref_eligibilite,
    });
  }
}
```

This function parses the uploaded `file` as `base64` format and update its `fileForm` value. If the user remove the `file`, the `eligibilite` of the `fileForm` will be reset to its `ref_eligibilite` (considered as a `const` in frontend) which is the original value gotten from backend.

For example, in the case of `REFUS_PIECE`, the current eligibilite is `refus piece`, the user (`club`) will upload another file to the frontend, the eligibilite will pass to `en attent FFF`. To undo the change, user will remove the file by clicking on `Cancel` after the file upload window pop up. The eligibilite will now return to its original value which is `refus piece`.

#### downloadFile

```ts
/**
 * WIP - can be improved by linking to storage on Azure Stack Edge
 * https://cdn-transverse.azureedge.net
 */
async downloadFile(file: any) {
  this.subs.sink = this.dossierService
    .requestDownloadFile(
      file.categorie,
      file.nature,
      file.azure_name ? file.azure_name : null
    )
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

This function is used to open a new tab that allows user to view the `PDF` file which is stored in `AzureStorage`. User can later decide whether or not to download the file.

#### spawnDossierObservableAndInitDossierFormAsSideEffects

```ts
/**
 * Create a `dossier$` stream. After subscribing to this stream,
 * a `dossierForm` building + some setup processes will be initialized as side effects
 */
private spawnDossierObservableAndInitDossierFormAsSideEffects() {
  return this.storageService.selectedDossier$.pipe(
    // side effects if subscribe to this stream
    tap((dossier: Dossier) => {
      if (dossier.criteres) {
        this.dossier = dossier;
        this.resetAllCritereArrays();
        // init dossierForm
        this.initInfoGenerale();
        this.initCriteresFormAndCriteresArrays(dossier.criteres);
        // setting ngClass object in html
        this.setCompetIdIcon();
        this.setDossierStatusColor();
        // init eligibleOfIncon$
        this.eligibleOfIncon$.next(
          this.dossierService.getInconEligibilite(this.dossierForm)
        );
        // init sumOfCumulable$
        this.sumOfCumulable$.next(
          this.dossierService.getSumOfCumulPoints(this.dossierForm)
        );
        // init totalPoints$
        this.totalPoints$.next(
          this.dossierService.getPointFromEligibilite(
            this.eligibleOfIncon$.value
          ) + this.sumOfCumulable$.value
        );
        // init aideFinanciere$
        this.aideFinanciere$.next(
          this.dossierService.calculAideFinan(
            this.storageService.competId,
            this.eligibleOfIncon$.value,
            this.totalPoints$.value
          )
        );
      }
    }),
    takeUntil(this.storageService.globalUnsubscribe)
  );
}
```

This function will create a `dossier$` stream. After subscribing to this stream, a `dossierForm` building + some setup processes will be initialized as side effects

#### subscribeEligibleFromDossier

```ts
/** Subscribe to `dossier$`, switch the stream to `Incontournables` Observable,
 * calculate the `eligibilite` in `Incontournables`, also update `totalPoints` and `aideFinanciere`
 */
private subscribeEligibleFromDossier(dossier$: Observable<Dossier>) {
  return dossier$
    .pipe(
      switchMap(() => {
        return this.dossierService.eligibleChangeObserver(this.dossierForm);
      })
    )
    .subscribe({
      next: () => {
        this.eligibleOfIncon$.next(
          this.dossierService.getInconEligibilite(this.dossierForm)
        );
        this.totalPoints$.next(
          this.dossierService.getPointFromEligibilite(
            this.eligibleOfIncon$.value
          ) + this.sumOfCumulable$.value
        );
        this.aideFinanciere$.next(
          this.dossierService.calculAideFinan(
            this.storageService.competId,
            this.eligibleOfIncon$.value,
            this.totalPoints$.value
          )
        );
      },
      error: (err: any) => {
        console.error(err);
      },
    });
}
```

This function subscribe to `dossier$` stream, then switch the stream to `Incontournables` observable, calculate the `eligibilite` in `Criteres Incontournables` field, it also updates `totalPoints` and `aideFinanciere`. This function will later be used in [setupDossierContent](#setupdossiercontent) function.

#### subscribeCumulableFromDossier

```ts
/** Subscribe to `dossier$`, switch the stream to `Cummulables` Observable,
 * calculate the `points` in `Cumulables`, also update `totalPoints` and `aideFinanciere`
 */
private subscribeCumulableFromDossier(dossier$: Observable<Dossier>) {
  return dossier$
    .pipe(
      switchMap(() => {
        return this.dossierService.pointsChangeObserver(this.dossierForm);
      })
    )
    .subscribe({
      next: () => {
        this.sumOfCumulable$.next(
          this.dossierService.getSumOfCumulPoints(this.dossierForm)
        );
        this.totalPoints$.next(
          this.dossierService.getPointFromEligibilite(
            this.eligibleOfIncon$.value
          ) + this.sumOfCumulable$.value
        );
        this.aideFinanciere$.next(
          this.dossierService.calculAideFinan(
            this.storageService.competId,
            this.eligibleOfIncon$.value,
            this.totalPoints$.value
          )
        );
      },
      error: (err: any) => {
        console.error(err);
      },
    });
}
```

This function subscribes to `dossier$` stream, then switch the stream to `Cummulables` observable, calculate the `points` in `Criteres Cumulables` field, it also updates `totalPoints` and `aideFinanciere`. This function will later be used in [setupDossierContent](#setupdossiercontent) function.

#### setupDossierContent

```ts
/**
 * Spawn `dossier$` and init `dossierForm` in its side effects.
 * Immediately subscribe to `eligibilite$` and `cumulables$` piped from `dossier$`
 * in order to calculate `incontournables` eligibilite, `cumulables` points and `totalPoints`
 */
setupDossierContent() {
  this.subs.sink = this.subscribeEligibleFromDossier(
    this.spawnDossierObservableAndInitDossierFormAsSideEffects()
  );
  this.subs.sink = this.subscribeCumulableFromDossier(
    this.spawnDossierObservableAndInitDossierFormAsSideEffects()
  );
}
```

This function will spawn a `dossier$` stream and init `dossierForm` in its side effects. Immediately after that, we switch the stream to `eligibilite$` stream and `cumulables$` stream in order to calculate `Criteres Incontournables` eligibilite, `Criteres Cumulables` points and `totalPoints`.

#### ngAfterViewChecked

```ts
ngAfterViewChecked(): void {
    this.cdRef.detectChanges();
  }
```

Because there are some proccesses such as setting competition icon, dossier statut color, etc... are initiated after the view is checked (because a `dossier` in `dossier$` stream could be updated at anytime after the view was checked, a http request could take a while to get the response). Therefore, we have to implement another change detection after the view is checked. Our application is just a small one so it does not impact much on the performance nor user experience.

#### ngOnDestroy

```ts
ngOnDestroy(): void {
  // unhide saisonBox
  this.storageService.showSaisonBox$.next(true);
  this.subs.unsubscribe();
  this.storageService.globalUnsubscribe.next();
  this.storageService.globalUnsubscribe.complete();
}
```

Upon destroying this component, we unhide the season selection dropdown box. We also terminate all the `dangling` observables (this is important, we did it to overcome the `child component duplication` bug)
