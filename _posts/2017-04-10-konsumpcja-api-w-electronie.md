---
layout:     post
title:      Konsumpcja API w Electronie i Angular 4
date:       2017-04-10 18:46
summary:    Consume API - czyli jak połączyć się z naszym API z Angulara.
tags:       DSP2017 Electron Angular4 RxJS

---

Consume API - czyli jak połączyć się z naszym API z Angulara.

## Angular 4 ##

Po pierwsze w celu wyjaśnienia. Mimo iż numeracja jest taka ciekawa, że mamy już Angulara 4 to tak na prawdę nie ma tutaj nic ciekawego. Napisane na nowszej wersji TypeScripta, rozszerzona dyrektywa ngIf i ngFor oraz poprawione animacji. A aktulizacja bardzo prosta:

```
npm update
```

I mamy :) A dokładniej to najlepiej jeśli zastosujemy się do instrukcji to powinniśmy zrobić to tak:

```
npm install @angular/common@latest @angular/compiler@latest @angular/compiler-cli@latest @angular/core@latest @angular/forms@latest @angular/http@latest @angular/platform-browser@latest @angular/platform-browser-dynamic@latest @angular/platform-server@latest @angular/router@latest @angular/animations@latest typescript@latest --save
```

Oczywiście zastosujmy tylko te paczki, których używamy. W przypadku tego projektu - wszystko zadziałało poprawnie.

## Do rzeczy - API ##

Dokładnie. Przejdźmy do rzeczy. Zaczynam jak zawsze od widoku. Jako iż delikatnie koncepcja rejestracj iisę zmieniła i utworzyła tak na prawdę podczas tworzenia API, przyszedł czas na utworzenie widoku.

### View first ###

```html
<div class="login-page">
    <h1>MassCo</h1>
    <span class="subtitle">Talk with coworkers around the world with MassCo!</span>

    <input placeholder="Cellphone number" type="tel" class="mc-input">

    <div class="step-container first-step">
        <button class="mc-button next-button">next</button>
    </div>

    <div class="step-container login-step">
        <label for="security-code">Provide security code from text message we send you:</label>
        <input placeholder="Enter security code" maxlength="4" type="number" class="mc-input" name="security-code">

        <button class="mc-button login-button">confirm</button>
    </div>

    <div class="step-container register-step">
        <input placeholder="Username" type="text" class="mc-input">
        <button class="mc-button register-button">create account</button>
    </div>

    <div class="step-container">
        <button class="mc-button next-button">back</button>
    </div>
</div>
```

Kilka zmian się pojawiło. Po pierwsze doszła zawartość do fragmentu z rejestracją. Pole **Username** oraz przycisk **create account**. Doszedł też nowy krok - potwierdzenie (**confirm**). Mimo iż jest to z poziomu widoku to samo - wymaganie narzuciła implementacja komponentu. Także przejdźmy do niej.

### login.service.ts ###

Najpierw kod - potem tłumaczenia:

``` js
import { Injectable } from '@angular/core';
import { Http, Headers, Response } from "@angular/http";
import { Observable } from "rxjs/Rx";

import { ConfirmVM, RegisterVM, ValidateVM } from './models';

import "rxjs/add/operator/do";
import "rxjs/add/operator/map";

@Injectable()
export class LoginService {
    API_URL: string;

    constructor(private http: Http) {
        this.API_URL = 'http://massco.api:56169/api';
    }

    validate(request: ValidateVM) {
        let url = this.API_URL + "/Account/validate";

        return this.http.post(url, request)
            .map(response => response.json())
            .catch(error => {
                return Observable.throw(error);
            });
    }

    createAccount(request: RegisterVM) {
        let url = this.API_URL + "/Account";

        return this.http.post(url, request)
            .map(response => response.json())
            .catch(error => {
                return Observable.throw(error);
            });
    }

    confirmAccount(request: ConfirmVM) {
        let url = this.API_URL + "/Account/confirm";

        return this.http.put(url, request)
            .catch(error => {
                return Observable.throw(error);
            });
    }

    authenticate(request : ConfirmVM) {
        let headers = new Headers();
        headers.append('Content-Type', 'application/x-www-form-urlencoded');

        let url = this.API_URL + "/token";
        let body = `accountId=${request.accountId}&confirmationCode=${request.confirmationCode}`

        return this.http.post(url, body, { headers: headers })
            .map(response => response.json())
            .catch(error => {
                return Observable.throw(error);
            });
    }
}
```

Do łatwiejszej implementacji wykorzystałem [reactive extensions for JS][1]. Bardzo przydatna biblioteka. Zaraz sami się przekonacie. Każdy serwis w angularze powinien być opatrzony dekoratorem **@Injectable()**. Dzięki temu serwis bedzie mozna wstrzyknąć do naszego komponentu. W konstruktorze wstrzykujemy zależność do Http oraz ustawiamy adres bazowy do API. potem kolejne metody. Generalnie każda jest bardzo podobna. Ustawiamy adres, wykonujemy posta, mapujemy odpowiedź z JSONa na obiekt i przechwytujemy wyjątek. Każda taka operacja jest właśnie z RxJS. Mapowanie - polega na odczytywaniu wyniku i zmapowanie na potrzebny przez nas wynik. Catch używany jest do przechwytywania błędnej odpowiedzi. Są jeszcze takie metody jak np. **do**, która umożliwia wykonanie określonej akcji. Warto generalnie zapoznać się z bilioteką RxJS i korzystać jak najwięcej - dużo praktycznych możliwości.

W metodzie Authenticate dodatkowo nasze API wymagało aby zmienić typ requesta. Możecie sami zobacyzć jak zmodyfikowane zostały nagłówki i jak przesłać poprawnie treść w takim wypadku.

### Komponent ###

PRzyszedł czas na niemalże ostatni krok. Jeśli chcemy użyć naszego nowo utworzonego serwisu musimy o tym powiadomić komponent za pomocą wpisu do tablicy **providers**:

```js
@Component({
    selector: 'login-app',
    templateUrl: './login.component.html',
    styleUrls: [ './login.component.less' ],
    providers: [
        LoginService
    ]    
})
```

Dodałem oczywiście też nowy krok, o którym wspomniałem i dodałem nowe modele do przechowywania i bindowania danych. Następnie kilka metod: login, register, confirm, checkPhone, confirmAccount. Jedna z metod wygląda np. tak:

```js
checkPhone() {
    var self = this;

    var request = new ValidateVM();
    request.phoneNumber = self.validateVM.phoneNumber;

    this.loginService
        .validate(request)
        .subscribe((data) => {
            self.confirmVm = new ConfirmVM();
            self.confirmVm.accountId = data.accountId;

            self.currentStep = self.steps.login;
        }, (error) => {
            // PhoneNumber NotFound
            if (error.status == 404) {
                self.registerVm = new RegisterVM();
                self.registerVm.phoneNumber = request.phoneNumber;

                self.currentStep = self.steps.register;
            }
        });       
}
```

Warto na początku zawsze pod kolakną zmienną przepisać wskazanie na *this*. Ze względu na to iż *this* wskazuje na inny obiekt w różnych kontekstach. Dzięki temu mamy pewność, ze będziemy mieli dostęp do komponentu w każdym momencie. Następnie przygotowujemy obiekt requestu i wywołujemy metodę **validate()** z **LoginService**. I tutaj ponownie przychodzi nam z pomocą RxJS i kolekcja Observable. Podpinamy się do wyniku za pomocą metody **subscribe()**, która jako pierwszy parametr przyjmuje akcję "pozytywną", a drugi parametr to akcja "błędu". W naszym wypadku akcja błędu dodatkowo jest obarczona logiką. 

Pozostałe metody są zaimplementowane w podobny sposób - możecie sami zobaczyć w [repozytorium][2].

### Sprawdzenie w działaniu ###

No dobrze. Przyszedł czas na pierwszą próbę. Uruchamiam. Wprowadzam numer do logowania i... **Not found**. WHAT?! No to ja szukam co złego jest w tym numerze telefonu. Sprawdzam bazę - ok, sprawdzam kod - ok, podłącam debuggera - nie wykonuje się! WTF? Postman - DZIAŁA! Szybkie googlanie - co może być nie tak? Co się okazuje - electron nie pozwala na łączenie z adresami lokalnymi (localhost, 127.0.0.1). No to szybko do pliku hosts za pomocą programu [Hosts File Editor][3] wprowadzam host name dla lokalnego adresu. Mój nowy adres to http://massco.api:56169/. 

Czas na kolejną próbę. Uruchamiam i znów... **Bad request**. Ok. Kolejna przeszkoda. Tym razem nie szukałem nawet przyczyny w kodzie. Próbuję w przeglądarce dostać się do dokumentacji mojego api... Wchodzę pod adres http://massco.api:56169/docs - a tutaj... Bad request! No ładnie. Także coś nie tak z ASP.NET Core. Niedoróbka - nie nadaje się na produkcję... ale szukam, szukam, szukam... IIS Express. Tutaj włąśnie leży problem. Jeśli chcemy koszystać z custom domain - musimy mieć uprawnienia admina. No cóż. Trzeba było zrobić restart VS z uprawnieniami administratora. Uruchamiam - działa! UF.

No i znów - kolejna próba. Czy tym razem pójdzie? Jest! Działa. Sam kod pisałem poniżej godziny. Pierwsze uruchomienie - dopiero po kolejnej godzinie. Takie to przygody z programowaniem :) Nauka na przyszłość i trzeba iść dalej. Logowania, rejestracja - wszystko działa.

## Co dalej? ##

W kolejnym poście odrobinę o testowaniu kontrollerów. A w kolejnym tygodniu - tworzenie głównej aplikacji z przechowywaniem kontekstu użytkownika. Do usłyszenia!


  [1]: https://github.com/Reactive-Extensions/RxJS
  [2]: https://github.com/duszekmestre/MassCo/commit/949814718e6d837c0d9dad80f1d17141539d2037
  [3]: https://scottlerch.github.io/HostsFileEditor/