---
layout:     post
title:      Angular 2.0 + Webpack i Electron
date:       2017-03-08 19:00
summary:    Angular 2 z Webpackiem dla mnie to wciąż wielka zmora. Na każdej stronie jest masa poradników, tutoriali ale niestety tak wiele z nich wymaga już troszkę większej wiedzy od osób (node, npm, ...). A w tym momencie nie dość, że ja tej wiedzy nie posiadam, to jeszcze próbuję to złączyć z Electronem! Zapraszam do lektury i przejscia ze mną tej przygody.
tags:       DSP2017 MassCo angular2 webpack electron
categories: dsp2017
---


Angular 2 z Webpackiem dla mnie to wciąż wielka zmora. Na każdej stronie jest masa poradników, tutoriali ale niestety tak wiele z nich wymaga już troszkę większej wiedzy od osób (node, npm, ...). A w tym momencie nie dość, że ja tej wiedzy nie posiadam, to jeszcze próbuję to złączyć z Electronem! Zapraszam do lektury i przejscia ze mną tej przygody.

## "Szybki" start z Angularem 2 ##

Na szczęście to jest najprostsza ścieżka. A przynajmniej tak mi się w tym momencie wydaje. Pamiętajcie, że piszę bloga zgodnie z zasadą live blogging. Czyli to co Wam opisuję jednocześnie robię :). Dlatego jeśli będą komplikacje po drodze - to dopiero na koniec tego posta zapewne będzie na repo najaktualniejszy i działający kod. Po pierwsze do dependencies w pliku packages.json dorzucam angulara:

``` json
"@angular/common": "^2.4.9",
"@angular/compiler": "^2.4.9",
"@angular/core": "^2.4.9",
"@angular/forms": "^2.4.9",
"@angular/http": "^2.4.9",
"@angular/platform-browser": "^2.4.9",
"@angular/platform-browser-dynamic": "^2.4.9",
"@angular/router": "^3.4.9",
```

a potem znana już nam metoda w command line:

```
npm install
```

Ha!. Zainstalowane. I to było to co najprostsze :) Było szybko? Było. Problem, że to jedynie instalacja. :( Ale cóż. Czas przejść dalej.

### Moduł aplikacji ###

Na początek postępuję z instrukcją przygotowania środowiska wg [strony angulara2][1]. Jedynym wyjątkiem jest to, że robimy to pod Webpacka. Ale wszystko zróbmy krok po kroku.

Po pierwsze jak w ostatnim wpisie wspomniałem po zalogowaniu/przejsciu ekranu logowania przenosimy się do nowego okna. Te okno buduję w tym momencie w inny sposób niż ekran logowania. Ale to też dlatego, że chcę to zrobić w pełni poprawnie (a przynajmniej tak myślę). Najpierw bootstrap aplikacji:

```js
import { platformBrowserDynamic } from '@angular/platform-browser-dynamic';

import { AppModule } from './app.module';

platformBrowserDynamic().bootstrapModule(AppModule);
```

Jak widzicie zaimportowaliśmy tutaj też nowy moduł (app.module). Nazwa modułu związana jest aktualnie z nazwą pliku, który nazywa się app.module.ts a jego zawartość to:

```js
import { NgModule }      from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { AppComponent }  from './app.component';

@NgModule({
  imports:      [ BrowserModule ],
  declarations: [ AppComponent ],
  bootstrap:    [ AppComponent ]
})
export class AppModule { }
```

Tutaj ponownie widzimy, że dodaję jako deklarację AppComponent i jest on jednocześnie komponentem rozruchowym (właściwość *bootstrap*). A oto jak wygląda komponent:

```js
import { Component } from '@angular/core';

@Component({
  selector: 'app',
  template: `<h1>Hello {{name}}</h1>`
})
export class AppComponent { 
    name = 'World'; 
}
```

No cóż. Szału nie ma :) Póki co to komponent jest całkiem czysty i nie zawiera żadnych wodotrysków. Teraz wystarczy go wykorzystać w pliku index.html:

```html
<app>Loading awesome MassCo app...</app>
```

Powyższy TAG dodajemy do body. Ten tekst ładujący dodajemy głównie po to aby podczas rozruchu aplikacji wyświetlił się jakiś tekst. Aczkolwiek aplikacja teraz jest tak lekka i mała, że po prostu nie zauważymy tego tekstu.

### Konfiguracja webpacka ###

W tym momencie aby to wszystko jeszcze zadziałało trzeba tylko skonfigurować webpacka. Lekko zmodyfikowałem wartości we właściwości entry:

```json
entry: {
    'login': './login/login',
    'login-vendor': './login/vendor',
    'app-polyfills': './app/polyfills',
    'app-vendor': './app/vendor',
    'app': './app/app',
}
```

A takze wartości plugins (usunięte elementy):

```json
plugins: [
    new CommonsChunkPlugin({ name: 'common', filename: 'common.js' })
]
```

W sumie nic nadzwyczajnego. Co do pierwszej zmiany. Wynikała ona z tego, ze chciałem powydzielać importy zewnętrzych modułów poza konfigurację webpacka. Polyfills i vendor to dwa plii (polyfills.ts i vendor.ts), których zawartość odpowiednio to:

```js
import 'zone.js/dist/zone';
import 'reflect-metadata';

if (process.env.ENV === 'production') {
    // Production
} else {
    // Development and test
    Error['stackTraceLimit'] = Infinity;
    require('zone.js/dist/long-stack-trace-zone');
}
```

```js
// RxJS
import 'rxjs';

// Angular
import '@angular/platform-browser';
import '@angular/platform-browser-dynamic';
import '@angular/core';
import '@angular/common';
import '@angular/http';
import '@angular/router';
```

Natomiast co do drugiej zmiany. Sam jej nie rozumiem jeszcze, dlatego nie wytłumaczę teraz :(. Może ktoś mógłby mi podpowiedzieć w ogóle co ona oznacza? Będę wdzięczny :)

### watch, start ... zwiększenie produktywności ###

Nie wiem, czy już o tym pisałem ale zawsze warto powtórzyć. Do pliku packages.json dodałem ostatnio wpis w części związanej ze skryptami:

```json
"watch": "webpack --watch --progress --profile --colors --display-error-details --display-cached"
```

Dzięki takiemu zabiegowi mozemy teraz uruchomić webpacka w trybie watch, czyli natychmiastowy rebuild po wykryciu zmian w plikach (nie wszystkich - angularowych). ALE. okazuje się, że nie mozemy wtedy uruchomić electrona z jednej konsoli bo webpack wciąż nasłuchuje. Oczywiście są dwie opcje. Można łączyć komendy i zrobić dwie operacja na raz. A druga opcja to uruchomić dwie konsole. A tam gdzie dwoje się bije trzeci korzysta. 

Jeśli korzystacie z Visual Studio Code tak jak ja to jest trzecia opcja. Terminal. Uruchamiamy ją za pomocą skrótu **Ctrl+\`**. Plusem jest to, że uruchamia się ona odrazu w kontekście projektu, w którym piszemy. Druga fajna opcja to możliwość uruchmiania kilku terminali (**Ctrl+Shift+\`**). I to jest właśnie najciekawsze. Bo możemy w jednym z nich uruchomić webpacka w trybie watch, w drugim uruchomić electrona i cieszyć się szybszymi zmianami. Po każdym zapisie i przebuildowaniu przez webpacka jedyne co musimy to w okienku aplikacji wykonać skrót **Ctrl+R**. I mozemy cieszyć się szybkimi zmianami. To jest też wielka zaleta samego pisania aplikacji desktopowych w tej technologii.


  [1]: https://angular.io/docs/ts/latest/guide/setup.html