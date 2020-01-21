---
layout:     post
title:      Angular 4 routing
date:       2017-04-25 18:00
summary:    W electronie działamy cały czas w kontekście strony internetowej. Takiego wielkiego SPA (Single Page Application). Oczywiście możemy robić różnorakie rzeczy i np serwować jedynie statyczny content HTML z przeładowaniem całej strony. ALE. Przecież mamy Angulara! Jak zrobić routing i co to takiego?
tags:       DSP2017 angular4 routing electron

---

W electronie działamy cały czas w kontekście strony internetowej. Takiego wielkiego SPA (Single Page Application). Oczywiście możemy robić różnorakie rzeczy i np serwować jedynie statyczny content HTML z przeładowaniem całej strony. ALE. Przecież mamy Angulara! Jak zrobić routing i co to takiego?

## Router ##

Heh. Takie urządzenie, które daje nam dostęp do internetu :) No niby tak :P Ale tak naprawdę pojęcie **router** oznacza kierowanie. Czyli gdy chcemy gdzieś użytkownika skierować to przechodzi to przez router i przenosi nas do odpowiedniego widoku. Tym dokładnie zajmuje się router/routing w angularze. Umożliwia nawigowanie z jednego widoku do drugiego jako konsekwencja podejmowanych przez użytkownika aktywności.

## Konfiguracja ##

Jak pamiętacie z poprzedniego [postu o planowaniu][1] - będziemy mieli kilka modułów w aplikacji. W związku z tym moim celem było nie tylko skonfigurowanie routingu ale także odpowiednie go podzielenia na poszczególne sekcje (moduły). Okazuje się, że to wszystko jest możliwe w Angularze 4 (i wcześniejszych wersjach). 

Na początek dodałem jedynie moduł do aktywności. 

```js
import { NgModule } from '@angular/core';

import { ActivityRoutingModule } from './activity-routing.module';

import { ActivitiesComponent } from './activity.component';

@NgModule({
    imports: [
        ActivityRoutingModule
    ],
    declarations: [
        ActivitiesComponent
    ],
    providers: []
})
export class ActivityModule { }
```

Widzimy tutaj znane nam już skłądowe angulara. NgModule - dekorator, który umożliwia oznaczenie klasy jako moduł aplikacji. W sekcji imports możemy dodać inne moduły, od których modułaktywności jest zależny. W tym wypadku jest to moduł routingu. W deklaracjach informujemy o komponentach, które udostępniamy w tym module. 

Jak natomiast wygląda konfigurowanie routingu? 

### Submodule - aktywności ###

Na początek skonfigurowałem go jedynie na poziomie modułu:

```js
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';

import { ActivitiesComponent } from './activity.component';

const activityRoutes: Routes = [
    {
        path: 'activity',
        component: ActivitiesComponent,
        data: {
            title: 'Activities'
        }
    }
];

@NgModule({
    imports: [
        RouterModule.forRoot(activityRoutes, { useHash: true })
    ],
    exports: [
        RouterModule
    ]
})
export class ActivityRoutingModule { }
```

Trzeba zaimportować koniecznie moduł **RouterModule** i klasę **Routes**. Następnie zdefiniowałem póki co jeden routing:

1. **path** - ścieżka do widoku
2. **component** - komponent odpowiedzialny za obsługę tej ścieżki
3. **data** - dodatkowe dane, które mozemy wykorzystać w późniejszym czasie

W sekcji *imports* dodałem inicjalizację routera z dość kontrowersyjną opcją **useHash**. Dlaczego kontrowercyjną? Generalnie wszystkie współczesne przeglądarki wspierają routingi w stylu HTML 5. Czyli ścieżki wirtualne, np: http://adres.www/activities jednakże jest z nimi pewien problem. One działają tylko, gdy korzystamy z protokołu HTTP/S. Ale nie działają, gdy korzystamy z "protokołu" File (file://). A w Electronie przecież tak jest! Dlatego musimy koniecznie wykorzystać opcję **useHash**, żeby obejść ten problem.

### Main module - APP ###

Konfiguracja routingu na poziomie podmodułu to dopiero początek. Teraz czas na główny moduł aplikacji i jego routing:

```js
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';

import { PageNotFoundComponent } from './core/page-not-found.component';

const appRoutes: Routes = [
    { path: '', redirectTo: '/activity', pathMatch: 'full' },
    { path: '**', component: PageNotFoundComponent }
];

@NgModule({
    imports: [
        RouterModule.forRoot(appRoutes, { useHash: true })
    ],
    exports: [
        RouterModule
    ]
})
export class AppRoutingModule { }
```

Teraz wspomnę tylko o czymś takim jak **'\*\*'**. Jest to wirtualna ścieżka, kówiąca o tym, że konfigurujemy każdą niedopasowaną ścieżkę. Co to nam daje? Możliwość obsługi błędu HTTP **404 Not Found**. Tutaj mogłem to zrobić w dwojaki sposób. Albo obsłużyć ten błąd albo przekierować na inny widok. Wybrałem pierwszą opcję. To da mi też możliwość zalogowania takiego przypadku na przyszłość i postaram się wtedy z logów odczytywać takie informacje i naprawię szybciej błąd.

>**!!! UWAGA !!!**
>
>**Dwie gwiazdki** **\*\*** dodajemy zawsze na końcu! Co jeśli tak nie będzie? Obsłuży ona wszystkie przypadki i nigdy nie dostaniemy innego widoku. W angularze routing działa na zasadzie - pierwsze dopasowanie zwycięża. Dlatego bardzo uważajmy na kolejność wpisów w tabeli routingu. Czyli zgodnie z zasadą: **Od szczegółu do ogółu**.

Kolejnym krokiem jest użycie routingów w głównym module:

```js
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { Router } from '@angular/router';

import { AppRoutingModule } from './app-routing.module';

import { ActivityModule } from './modules/activity/activity.module';

import { AppComponent } from './app.component';
import { PageNotFoundComponent } from './core/page-not-found.component';

@NgModule({
  imports: [
    BrowserModule,
    ActivityModule,
    AppRoutingModule
  ],
  declarations: [
    AppComponent,
    PageNotFoundComponent
  ],
  bootstrap: [AppComponent]
})
export class AppModule {
  constructor(router: Router) {
    console.log('Routes: ', JSON.stringify(router.config, undefined, 2));
  }
}
```

Tutaj również warto zwrócić uwagę na kolejność dodawania modułów w sekcji *imports*. Ciekawą i dość przydatną diagnostycznie rzeczą jest to, co widzicie w konstruktorze. Umożliwa to podejrzenie w konsoli całej tablicy routingu.

### Router na widoku ###

No tak. Mamy skonfigurowane już praktycznie wszystko, czego potrzebujemy na początek. Co dalej? Trzeba to jakoś pokazać na widoku:

```html
<section id="body" class="full-width">
    <aside id="sidebar" class="column-left">
        <header>
            <div class="app-logo">
                <div></div>
                <div></div>
                <div></div>
            </div>

            <h1>MASS.CO</h1>
        </header>

        <nav id="main-nav">
            <ul>
                <li><a routerLink="/activity" routerLinkActive="active">Activity</a></li>
            </ul>
        </nav>
    </aside>

    <section id="content" class="column-right">
        <router-outlet></router-outlet>
    </section>
</section>
```

Dwa istotne elementy to **routerLink** oraz **router-outlet**. Pierwszy z nich umożliwia dodanie klikalnego przycisku bądź linka. Podajemy tam ścieżkę. Ciekawym elementem jest też **routerLinkActive**. Dzięki niemu - element, który ma ten fragment zostanie udekorowany klasą podaną jako parametr (w moim wypadku *active*), jeśli będzie aktualnie na danym widoku. 

**router-outlet** to komponent, który jest wskazaniem dla mechanizmu routingu, gdzie ma umieścić treść określonego widoku. Jest to tak jakby placeholder. **UWAGA** - treść widoku bedzie umieszczona jako kolejny element, a nie wewnątrz niego. Dlatego nie możemy w taki sposób stylować poszczególnych elementów.

Fajny mjest też to, że konfigurując w taki sposób możemy zagnieżdżać router-outlety. O tym za chwilę.

### uwagi ###

Mimo tego co piszą w tutorialach angulara itp. W Electronie musimy uważać na taki zabieg:

```html
<base href="/" />
```

Nie powinniśmy go używać. Dlaczego? Bo używamy **useHash: true**. Warte zapamietania.

## Child route ##

Czasami jest taka potrzeba (w sumie to często), żeby wyświetlać widok w widoku. I nie mówię tutaj o komponentach ale o dynamicznie zmienianym widoku wewnątrz innego widoku. Czyli mówiąc po angularowemu: router inside router. Pokaże to na przykłądzie innego modułu:

```js
const teamsRoutes: Routes = [
    {
        path: 'teams',
        component: TeamsComponent,
        data: {
            title: 'Teams'
        },
        children:
        [
            { path: 'discover', component: DiscoverTeamsComponent },
            { path: 'new-team', component: NewTeamComponent, outlet: 'teams-popup' },
            { path: '', component: MyTeamsComponent }
        ]
    }
];
```

Powyżej jedynie kawałem samej definicji routingu. Widzimy tutaj, że mamy możliwość zdefiniowania dzieci dla routingu. Są to po prostu kolejne ścieżki, które będą wyświetlane w podrouterze, np: **#/teams/discover**. Przykład tego jak może zmieniać się routing możecie zobaczyć na plunkerze [angular routing live demo][2]. 

### named outlets ###

A co jeśli chcemy mieć dwa routery na tej samej stronie? I tutaj własnie pojawia się nowość w angularze (od wersji Angular 2): **named outlets**. Jak to zrobić? JEśli macie wprawne oko to ścieżka **#/teams/new-team** posiada parametr *outlet*, który wskazuje router-outlet, w którym ma się wyświetlić ten widok. A w HTML wygląda to tak:

```html
<nav class="mc-tabs">
    <ul>
        <li>
            <a routerLink="./" routerLinkActive="active" [routerLinkActiveOptions]="{ exact: true }">My teams</a>
        </li>
        <li>
            <a routerLink="./discover" routerLinkActive="active">Discover</a>
        </li>
    </ul>

    <a [routerLink]="[{ outlets: { 'teams-popup': ['new-team'] } }]" class="new-team">
        <i class="horizontal"></i>
        <i class="vertical"></i>
    </a>
</nav>

<router-outlet></router-outlet>
<router-outlet name="teams-popup"></router-outlet>
```

Zauważyć można tutaj dwa nowe elementy: więcej parametrów w **routerLink** oraz nazwę w **router-outlet**. Dzięki mniej więcej takiemu zabiegowi możemy po naciśnięciu przycisku otworzyć widok w drugim routerze. Osobiście uważam, że troszkę to dziwnie wygląda, ze nie dość, ze podaliśmy w deklaracji routingu nazwę outletu, to jeszcze podczas wywoływania też musimy to podać. *ALE CÓŻ*. Tak po prostu trzeba. 

## Tytuł strony - HACK ##

No niestety to jest hack. Po chwilach przeszukiwania internetu znalazłem taką możliwość aby ustawiać tytuł strony na podstawie parametru w routingu. Jak? Proszę bardzo:

```js
import { NgModule } from '@angular/core';
import { BrowserModule, Title } from '@angular/platform-browser';
import { Router, ActivatedRouteSnapshot, NavigationEnd } from '@angular/router';

import { ActivityModule } from './modules/activity/activity.module';
import { TeamsModule } from './modules/teams/teams.module';

import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';

import { PageNotFoundComponent } from './core/page-not-found.component';

@NgModule({
  imports: [
    BrowserModule,
    ActivityModule,
    TeamsModule,
    AppRoutingModule
  ],
  declarations: [
    AppComponent,
    PageNotFoundComponent
  ],
  providers: [
    Title
  ],
  bootstrap: [AppComponent]
})
export class AppModule {
  constructor(router: Router, title: Title) {
    console.log('Routes: ', JSON.stringify(router.config, undefined, 2));

    router.events.subscribe((event) => {
      if (event instanceof NavigationEnd) {
        console.log(event);
        
        var titleToSet = this.getDeepestTitle(router.routerState.snapshot.root);
        title.setTitle(titleToSet);
      }
    });
  }

  private getDeepestTitle(routeSnapshot: ActivatedRouteSnapshot) {
    var title = routeSnapshot.data ? routeSnapshot.data['title'] : '';
    if (routeSnapshot.firstChild) {
      title = this.getDeepestTitle(routeSnapshot.firstChild) || title;
    }
    return title;
  }
}
```

Jest to kod głównego modułu aplikacji. Angular daje dostęp do tytułu strony za pomocą [TitleProvider][3]. Musimy go dodać do sekcji *providers* w dekoratorze **NgModule**. Potem w konstruktorze podpinamy się pod event routera. Musimy sprawdzić, czy jest to event *NavigationEnd* i tam pobrać z danych routingu nazwę, którą ustawiliśmy w deklaracji routingu. Nie miłe - ale działa i to dość sprawnie. Ja dodatkowo dodałem sobie logowanie eventu w konsolu w celach diagnostycznych, co szczerze każdemu polecam na etapie programowania.

## Podsumowanie ##

Jak widzicie - troszkę tego wszystkiego jest. A w sumie to nie są jeszcze wszystkie możliwości. Odsyłam Was do [dokumentacji][4], gdzie możecie poznać wszystkie tajniki routingu. Jest tam generalnie wszystko. Ja większość powyższych rzeczy wziąłem z plunkera, o którym wyżej pisałem + kilka szczegółów z dokumentacji. Powyższe wypociny to również jedynie drobna część całości kodu, który ostatni napisałem. Po więcej zapraszam na stronę pojektu [MassCo on GitHub][5]. 

## Co dalej? ##

Generalnie zajmuję się teraz tworzeniem widoku zespołów. Zajmie to pewnie troszkę czasu. Ale mimo wszystko postaram się coś ciekawego w tym wszystkim znaleźć i opisać Wam. Jakbyście mieli kiedykolwiek do mnie jakieś pytania to możecie pisać do mnie maila lub za pośrednictwem [Facebooka][6]. Do usłyszenia!


  [1]: /dsp2017/2017/04/21/planowanie/
  [2]: https://angular.io/resources/live-examples/router/ts/eplnkr.html
  [3]: https://angular.io/docs/ts/latest/api/platform-browser/index/Title-class.html
  [4]: https://angular.io/docs/ts/latest/guide/router.html
  [5]: https://github.com/duszekmestre/MassCo
  [6]: https://www.facebook.com/duszekarciszewski/