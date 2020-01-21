---
layout:     post
title:      Webpack i Angular 2 w Electronie
date:       2017-03-01 21:00
summary:    Ten temat to jeden z trudniejszych jakie musiałem ogarnać. Mimo iż kilkukrotnie przeoribem tutoriale to nadal czuję się ciemny w tym temacie. Jednak dzisiaj razem z Wami spróbuję krok po kroku przejść przez doinstalowanie webpacka i angulara 2 do naszej aplikacji w Electronie.
tags:       DSP2017 MassCo webpack angular2 electron

---

Ten temat to jeden z trudniejszych jakie musiałem ogarnać. Mimo iż kilkukrotnie przeoribem tutoriale to nadal czuję się ciemny w tym temacie. Jednak dzisiaj razem z Wami spróbuję krok po kroku przejść przez doinstalowanie webpacka i angulara 2 do naszej aplikacji w Electronie.

## Let's begin ##
Na początek zatualizujemy (dev)dependencies w pliku package.json:

```json
"devDependencies": {
    "electron-prebuilt": "^1.4.13",
    "ts-loader": "^2.0.1",
    "typescript": "^2.2.1",
    "webpack": "^2.2.1",
    "webpack-dev-server": "^2.4.1"
},
"dependencies": {
    "angular2": "^2.0.0-beta.21",
    "es6-shim": "^0.35.3",
    "reflect-metadata": "^0.1.10",
    "rxjs": "^5.2.0",
    "zone.js": "^0.7.7"
}
```

Po zapisaniu pliku wystarczy wywołać komendę: 

```
npm install
```

Dzięki temu npm pobierze nam paczki w wersjach, których potrzebujemy. Kolejnym krokiem bedzie skonfigurowanie TypeScript, którego będę używał do pisania w Angularze. (dla tych, którzy nie wiedzą - TypeScript to w sumie taki JavaScript ale silnie typowany i sprawdza statycznie, czy nie ma błędów w kodzie. Taki jakby język programowania). Aby skonfigurować TypeScript nalezy w głownym katalogu utworzyć plik tsconfig.json z mniej więcej taką zawartością:

```json
{
    "compilerOptions": {
        "target": "ES5",
        "module": "commonjs",
        "removeComments": true,
        "emitDecoratorMetadata": true,
        "experimentalDecorators": true,
        "sourceMap": true
    },
    "files": [
        "login/login.ts"
    ]
}
```

No i teraz na koniec smaczki :) Konfiguracja WebPacka w pliku webpack.config.js:

```js
// webpack.config.js
var path = require('path');
var webpack = require('webpack');

var CommonsChunkPlugin = webpack.optimize.CommonsChunkPlugin;

module.exports = {
    devtool: 'source-map',
    entry: {
        'angular2': [
            'rxjs',
            'reflect-metadata',
            'angular2/core',
            'angular2/router',
            'angular2/http'
        ],
        'login': './login/app'
    },
    output: {
        path: __dirname + '/build/',
        publicPath: 'build/',
        filename: '[name].js',
        sourceMapFilename: '[name].js.map',
        chunkFilename: '[id].chunk.js'
    },
    resolve: {
        extensions: ['.ts', '.js', '.json', '.css', '.html']
    },
    module: {
        loaders: [{
            test: /\.ts$/,
            loader: 'ts-loader',
            exclude: [/node_modules/]
        }]
    },
    plugins: [
        new CommonsChunkPlugin({ name: 'angular2', filename: 'angular2.js', minChunks: Infinity }),
        new CommonsChunkPlugin({ name: 'common', filename: 'common.js' })
    ]
};

```

Taka konfiguracja jak powyżej spowoduje, że dostaniemy dwa bundle: 

 - *angular2.js* (wszystko co związane z angularem)
 - *login.js* - nasza aplikacja okna logowania

A teraz już dodamy component login-app w pliku login/app.ts:

```js
///

import { bootstrap } from 'angular2/platform/browser';
import { Component } from 'angular2/core';

@Component({
    selector: 'login-app',
    template: '<h1>Hello World from login window!</h1>'
})

export class AppComponent { }

bootstrap(AppComponent);
```

Teraz już tylko modyfikacja pliku login/index.html:

```html
<html>

<head>
    <title>Login</title>
</head>

<body>
    <login-app></login-app>

    <script src="../node_modules/angular2/bundles/angular2-polyfills.js"></script>
    <script src="../build/common.js"></script>
    <script src="../build/angular2.js"></script>
    <script src="../build/login.js"></script>
</body>

</html> 
```

## Uruchamiamy ##

Teraz już na koniec wystarczy tylko uruchomić dwa polecenie:

```
npm run build
npm start
```

I naszym oczom ukaże się: 

![Hello Angular][1]

## Co dalej? ##

Kolejnym krokiem bedzie dodanie styli i ekranu logowania (HTML). W razie pytań komentarze. Do usłyszenia!


  [1]: {{ site.baseurl }}/images/HelloAngular.PNG