# CLUB-ACCUEIL

The club homepage of the LCF application.

## Description:

The club homepage doesn't have much features. It currently has a simple text wall and a clickable sticker which directs us to [CLUB-CANDIDATURE](./club-candidature.md) page

## Files:

```ignore
club-accueil.component.html
club-accueil.component.scss
club-accueil.component.ts
```

## Functionalities:

### General:

```ts
ngOnInit() {
    this.checkSaNoFromUrl();
    this.title.setTitle(UtilsService.createTitle('Votre Projet Club'));

    this.sub = this.storageService
    .selectedSaisonSaNo$
    .pipe(takeUntil(this.storageService.globalUnsubscribe))
    .subscribe((sa_no: number) => {
    this.selectedSaisonSaNo = sa_no;
    // TODO: do something in this page
    });
}
```

When we open this page, it will check the `season` string in the `url` is indeed in range of `seasons` defined in `user.saisons` ([checkSaNoFromUrl](#checksanofromurl)). Then it stores the `season` value in `storage` . Else, redirect to [PAGE-NOT-FOUND](../shared/page-not-found.md) if the `season` is out of range. The `selectedSaisonSaNo` variable later is used to correctly direct user to the next page upon clicking on the sticker.

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

#### clickCandidature

```ts
clickCandidature(): void {
    this.storageService.firstMenuItemActivated = false;
    this.storageService.secondMenuItemActivated = true;
  }
```

This function allows the second menu-item to be highlighted and dehighlight the first one.

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
<div class="content-home">
  <div class="content">
    <h1>Votre Projet Club</h1>
    <p>
      Votre club est interess?? par une demande d'accompagnement et de
      structuration de son ?? Projet Club ??. Pour aller plus loin dans cette
      d??marche, vous pouvez remplir votre autodiagnostic au travers des
      diff??rents Labels propos??s par la F??d??ration Fran??aise de Football.
      <br /><br />
      En entrant dans cette d??marche, votre Ligue et ses Districts (??lus et
      Conseillers Techniques) s'engagent ?? vous rencontrer et vous guident pour
      d??velopper votre projet club.
      <br /><br />
      En fonction des vocations de votre club, vous pouvez alors orienter vers
      un diagnostic du Label qui vous correspond. Bien ??videmment, cet
      autodiagnostic n'est pas r??serv?? aux clubs qui souhaitent obtenir un
      label. Il est ?? votre disposition pour faire le point sur votre
      structuration et d??buter une d??marche d'accompagnement avec votre
      territoire.
      <br /><br />
      Pour les clubs nationaux amateurs, il est ??galement possible de faire une
      demande d'obtention de la Licence ?? Club F??d??ral ??.
    </p>
    <hr />
    <div class="vignettes">
      <div class="label">
        <a
          [routerLink]="[selectedSaisonSaNo ? '/club/' + selectedSaisonSaNo + '/lcf' : '/club/']"
          (click)="clickCandidature()"
        >
          <div>
            <span class="tit">LICENCE ?? CLUB F??D??RAL ??</span>
          </div>
        </a>
      </div>
    </div>
  </div>
</div>
```
