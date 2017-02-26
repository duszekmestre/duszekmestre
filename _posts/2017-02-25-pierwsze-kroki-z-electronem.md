---
layout:     post
title:      Pierwsze kroki z Electronem
date:       2017-02-25 20:30
summary:    Tworzymy aplikację na electronie! Dzisiaj krok po kroku przeprowadzę Was po krokach do tworzenia prostej aplikacji. Celem na dziś jest utworzenie aplikacji desktopowej z użyciem Angular 2.
tags:       DSP2017 MassCo
categories: dsp2017
---

Tworzymy aplikację na electronie! Dzisiaj krok po kroku przeprowadzę Was po krokach do tworzenia prostej aplikacji. Celem na dziś jest utworzenie aplikacji desktopowej z użyciem Angular 2.

## Prerequisites - czyli pre... wymagania wstępne :P ##
Od czego zacząć? Po pierwsze musimy mieć zainstalowany node.js, który możemy pobrać [tutaj](https://nodejs.org/en/).
Po zainstalowaniu upewnijmy się, że mamy do niego dostęp. Wystarczy uruchomić wiersz poleceń i wpisać polecenie:
```
npm -v
```

W moim przypadku wersja NPM to 3.10.10. Aby mieć pewność, że posiadamy najaktualniejsza wersję wystarczy wykonać komendę:

```
npm install npm@latest -g
```

Parametr *-g* oznacza, że zostanie zainstalowane globalnie, dzięki czemu będziemy mieli dostęp do komendy z wiersza poleceń.

Co gdy nie działa? :) No cóż. Strona Node lub [NPM](https://www.npmjs.com/) powinna być pomocna (ewentualnie stary, dobry wujek Google).

## Pierwsze kroki - instalacja electron ##
Instalacja jest prosta, krótka i dla tych, którzy mieli kiedykolwiek do czynienia z Node.js banalna i standardowa. Tworzymy katalog gdzie będziemy tworzyć naszą aplikację. W wierszu poleceń przechodzimy do katalogu docelowej aplikacji i wykonujemy polecenie:
```
npm init
```
Tutaj wypełniamy kreator. Jak mamy wątpliwosci to po prostu Enter. Z czasem i tak wszystko się wyprostuje. Kolejny krok to już electron.
```
npm install electron-prebuilt --save-dev
```
I już! Instalacja zakończona. I co dalej? Proponuję zajrzeć do pliku packages.json. To taki plik konfiguracyjny dla naszego projektu. U mnie wygląda on tak:
```json
{
    "name": "massco-desktop",
    "version": "0.1.0",
    "description": "Massive communication for corporate usage",
    "main": "index.js",
    "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1",
        "start": "electron ."
    },
    "repository": {
        "type": "git",
        "url": "git+https://github.com/duszekmestre/MassCo.git"
    },
    "keywords": [
        "messenger",
        "communicator",
        "corporate",
        "social",
        "docs",
        "preview"
    ],
    "author": "duszekmestre",
    "license": "ISC",
    "bugs": {
        "url": "https://github.com/duszekmestre/MassCo/issues"
    },
    "homepage": "https://github.com/duszekmestre/MassCo#readme",
    "devDependencies": {
        "electron-prebuilt": "^1.4.13"
    }
}
```
Kilka ważnych rzeczy na początek.

 - **name** - nazwa naszej aplikacji
 - **main** - plik uruchomieniowy, tutaj zacznie się życie naszej aplikacji
 - **devDependencies** - zależności naszego projektu, których potrzebujemy na etapie programowania
 - **dependencies** - (na tym etapie ich jeszcze nie ma) zlaeżności naszej aplikacji wymagane do uruchomienia jej
 - **scripts** - skryptu, które możemy definiować
 
Jeden ze skryptów dodałem już na początek. Taka linijka:
```json
"start": "electron ."
```
Dzięki temu będzie nam łatwo uruchomić naszą aplikację podczas programowania.

## Hello World! ##
No dobrze. Ale to nie wszystko. Przecież jakoś taka aplikacja musi działać! Co zrobić? Nasz kolejny krok to utworzenie pliku index.js (plik musi się nazywać dokładnie tak jak podało się w main w packages.json. W tym pliku zainicjalizujemy pierwsze okienko naszej aplikacji:
```js
const { app, BrowserWindow } = require('electron');

let startScreen;

app.on('ready', () => {
    startScreen = new BrowserWindow({
        width: 400,
        height: 700
    });

    startScreen.on('closed', () => {
        startScreen = null;
    });

    startScreen.loadURL('file://' + __dirname + '/login/index.html');
});

app.on('window-all-closed', () => {
    if (process.platform !== 'darwin') {
        app.quit();
    }
});
```

To teraz szykie podsumowanie. Na początek importujemy moduły z paczki electron. Następnie definiujemy globalny handler głównego okienka. Podpinamy się pod dwa eventy (zdarzenia). Pierwszy z nich po załadowaniu się programu otwiera okno aplikacji o wielkości 400 x 700 oraz ładuje "stronę" startową. W naszym wypadku będzie do okno logowania. Jednocześnie wskazujemy, że po zamknięciu okna ma się wyczyścić. Drugi ważny event to 'window-all-closed' - czyli zamknięcie wszytskch okien i wyłączenie aplikacji. W tym momencie zamykamy po prostu aplikację.

W pliku index.html nie pojawia się za to nic a nic nadzwyczajnego. Tworzymy po prostu najzwyklejszą stronę html:
```html
<html>
<head>
    <title>Login</title>
</head>

<body>
    <h1>Hello World</h1>
</body>
</html>
```

## Uruchomienie ##
A teraz najważniejsze. Czy wszystko się uda już teraz, za pierwszym razem? sprawdźmy. Skoro wciąż mamy otwarte okno wiersza poleceń - wystarczy jeśli wpiszemy komendę:
```
npm start
```
I oto użymy piękne okno: 

![HelloWorld][1]

## c.d.n. ##
Tym wpisem zakończę pierwsze kroki z electronem. W kolejnym wpisie dodamy do tego wszystkiego Angulara 2 i webpack. Wszystko wyjaśnię kolejnym razem i zachęcam do śledzenia wpisów! :)
Wszystkie źródła do kodu, który będzie rósł razem z wpisami można zobaczyć pod w moim repozytorium: [https://github.com/duszekmestre/MassCo][1]


  [1]: {{ site.baseurl }}/images/HelloWorld.PNG
