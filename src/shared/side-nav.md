# SIDE-NAV

The side navigation bar of the LCF application.

## Description:

This sidenav always appears on the left of the application. There are two versions of the sidenav, one for [CLUB](../lcf-club.md) and one for [DCN](../lcf-dcn.md)

## Files:

```ignore
side-nav.component.html
side-nav.component.scss
side-nav.component.ts
```

## Functionalities:

### General:

```ts
ngOnInit(): void {
    if (this.storageService.connectId === 'dcn') {
      this.showClubSideNavBar = false;
    } else {
      this.showClubSideNavBar = true;
    }

    this.sub = this.storageService.getSelectedSaisonSaNo$.subscribe(
      (sa_no: number) => {
        this.selectedSaisonSaNo = sa_no;
      }
    );
}
```

Depending on who connected to the application ([CLUB](../lcf-club.md) or [DCN](../lcf-dcn.md)), show the corresponding `side-nav` version

### Specific:

```ts
clickFirstEl() {
    this.storageService.firstMenuItemActivated = true;
    this.storageService.secondMenuItemActivated = false;
}

clickSecondEl() {
    this.storageService.firstMenuItemActivated = false;
    this.storageService.secondMenuItemActivated = true;
}

resetAllEl() {
    this.storageService.firstMenuItemActivated = false;
    this.storageService.secondMenuItemActivated = false;
}
```

Highlight an item in `side-nav` upon clicking on it.

### HTML:

```html
<div class="container h100">
  <div class="colG" *ngIf="showClubSideNavBar">
    <ul>
      <li class="nv0">
        <a [routerLink]="['/club',selectedSaisonSaNo]" (click)="clickFirstEl()"
          >Projet Club <span></span
        ></a>
        <ul class="nav">
          <li class="nav-accueil">
            <a
              [ngClass]="{'active': storageService.firstMenuItemActivated$ | async}"
              [routerLink]="['/club', selectedSaisonSaNo]"
              (click)="clickFirstEl()"
              >Accueil</a
            >
          </li>
          <li class="nav-licence">
            <a
              [ngClass]="{'active': storageService.secondMenuItemActivated$ | async}"
              [routerLink]="['/club', selectedSaisonSaNo,'lcf']"
              (click)="clickSecondEl()"
              >Licence « Club Fédéral »</a
            >
          </li>
        </ul>
      </li>
    </ul>
  </div>

  <div class="colG nav2" *ngIf="!showClubSideNavBar">
    <ul class="vl margTop10">
      <li class="nv0 nva">
        <a
          [routerLink]="['/dcn',selectedSaisonSaNo,'lcf']"
          (click)="clickFirstEl()"
          >Licence « Club Fédéral » <span></span
        ></a>
        <ul class="nav">
          <li class="nav-search">
            <a
              [ngClass]="{'active': storageService.firstMenuItemActivated$ | async}"
              [routerLink]="['/dcn',selectedSaisonSaNo,'lcf']"
              (click)="clickFirstEl()"
              >Recherche / Liste</a
            >
          </li>
          <li class="nav-settings">
            <a
              [ngClass]="{'active': storageService.secondMenuItemActivated$ | async}"
              [routerLink]="['/dcn',selectedSaisonSaNo,'lcf','params']"
              (click)="clickSecondEl()"
              >Paramètres</a
            >
          </li>
        </ul>
      </li>
    </ul>
  </div>
  <router-outlet></router-outlet>
</div>
```
