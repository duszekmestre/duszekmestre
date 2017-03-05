---
layout:     post
title:      View First - login screen
date:       2017-03-05 17:00
summary:    Długo myślałem nad tym jakby miał taki ekran logowani wyglądać. Hasła są kiepskim pomysłem. Bo ile można, no nie? I dlatego chcę wprowadzić two-factor authentication. Ale najpierw... Design.
tags:       DSP2017 MassCo HTML CSS angular2 electron
categories: dsp2017
---

Długo myślałem nad tym jakby miał taki ekran logowani wyglądać. Hasła są kiepskim pomysłem. Bo ile można, no nie? I dlatego chcę wprowadzić two-factor authentication. Ale najpierw... Design.

## Dlaczego View First? ##

Nie wiem, czy taka nazwa istnieje w słowniku IT. Ale w moim tak. View first jest wg mnie podejściem idealnie dopasowanym do szybkich kroczków. Najpierw tworzę widok jakiego oczekuje. Oprogramowuję ekrany i następnie dopisuję logikę. To prawie jak Test First. W tym, że tutaj rolę takiego testu pełni widok. Aplikacja klienta będzie online'owa. Dziwne byłoby gdyby było inaczej. W sumie to messenger :) Zdecydowana większość logiki biznesowej bedzie po strone back-endu - części serwerowej.

## Krok po kroku ##

Także aby długo nie rozpisywać się w tej kwestii to opiszę kolejne kroki.

### Widok HTML ###

Po pierwsze to trzeba dodać widok. W ostatnim [commicie][1] pozmieniałem lekko nazwy plików itp, ale to nie jest najważniejsze. W naszym komponencie logowania *login-app* dodałem odwołania do template oraz styles. Po tych zmianach deklaracja komponentu wygląda tak:

```js
@Component({
    selector: 'login-app',
    templateUrl: './login.html',
    styleUrls: [ './login.css' ]    
})
```

Zaplanowałem, zeby nasz widok wyglądał tak:

![Login screen][4]

No i do tego dążyłem. HTML wygląda więc tak:

```html
<div class="login-page">
    <h1>MassCo</h1>
    <span class="subtitle">Talk with coworkers around the world with MassCo!</span>

    <input [disabled]="currentStep !== steps.first" [(ngModel)]="loginVm.phone" class="mc-input" type="tel" placeholder="Cellphone number">

    <div class="step-container first-step" *ngIf="currentStep === steps.first">
        <button class="mc-button next-button" (click)="checkPhone()">next</button>
    </div>

    <div class="step-container login-step" *ngIf="currentStep === steps.login">
        <label for="security-code">Provide security code from text message we send you:</label>
        <input [(ngModel)]="loginVm.confirmCode" name="security-code" class="mc-input" type="number" maxlength="4" placeholder="Enter security code">

        <button class="mc-button login-button" (click)="login()">login</button>
    </div>

    <div class="step-container register-step" *ngIf="currentStep === steps.register">
        <button class="mc-button register-button">register</button>
    </div>
</div>
```

Postanowiłem, ze wszystkie moje ogólne klasy CSS będą miały prefix *mc-*. Ale jak sami widzicie pojawiło się też kilka rzeczy "dziwnych". To są atrybuty angulara. Oto i one:

 - [disabled] - jak sama nazwa wskazuje, wskazujemy w jakich warunkach element ma być zablokowany
 - [(ngModel)] - two-way binding. Wpisujemy tutaj do jakiego modelu ma się zapisywać wartość pola (lub ma się ona odczytywać).
 - (click) - podpięcie pod event, czyli wskazujemy jaka akcja ma się odbyć po określonym evencie (tutaj click)
 - *ngIf - warunkowe wyświetlenie elementu. Wewnątrz wpisujemy warunek, dla którego element bedzie wrzucony w HTML.
 
Ale skad się biorą te wszystkie zmienne?

### Logika komponentu ###

Najłatwiej pisać mi kodem, dlatego od niego zacznę i wszystko objaśnię:

```js
export class LoginComponent { 
    steps = {
        first: 1,
        login: 2,
        register: 3
    };

    currentStep = this.steps.first;

    loginVm: LoginVM;
    registerVm: RegisterVM;

    constructor() {
        this.loginVm = new LoginVM();
        this.loginVm.phone = '500112233';
        this.loginVm.confirmCode = 123;
    }

    checkPhone() {
        var self = this;

        isValid({
            phone: this.loginVm.phone || '',
            country: 'pl'
        }, function(err, result) {
            if(result.isValid) {
                self.checkPhoneExists(self);
            } else {
                alert('Wrong phone number');
            }
        });        
    }

    checkPhoneExists(self: LoginComponent) {
        var exists = true;
        if(exists) {
            self.currentStep = self.steps.login;
        } else {
            self.registerVm = new RegisterVM();
            self.registerVm.phone = self.loginVm.phone;

            self.currentStep = self.steps.register;
        }
    }

    login() {
        if(this.loginVm.confirmCode <= 0) {
            alert('Wrong confirmation code');
        }

        // TODO: Verify security code

        this.openMainWindow();
    }

    openMainWindow() {
        electron.ipcRenderer.send('open-main-window');
    }
}
```

Nasz komponent na początek ma zadeklarowane zmienne. Pierwszą z nich traktuję jako pomocniczą dla czytelności kodu. Taki enum. Potem określam jaki jest aktualny krok ekranu, model logowania i rejestracji. 

Dla osób, które programowały w językach obiektowych pojecie konstruktora bedzie znane. To taka specjalna metoda, która wykonuje sie po utworzeniu instancji obiektu (w tym wypadku komponentu). W TypeScript ten konstruktor nazywa sie po prostu constructor(). Moze być z parametrami lub bez (tutaj bez). Wewnątrz niego inicjalizuję wartości zmiennych. Dodatkowo widać, zę zainicjalizowałem model logowania. Póki nie mam całej części serwerowej odpowiedzialnej za logowanie dużo szybsze jest mockowanie danych do widoków. 

Następnie mamy 4 metody. W sumie 2 to są takie metody wejściowe, które bindujemy (łączymy określoną akcję z widokiem) do widoku. checkPhone() i login(). Pierwsza z nich odpowiada za walidację numeru telefonu. Dzięki wykorzystaniu biblioteki *phonelib* mamy możliwość zwalidowania numeru telefonu. Dzięki tej bibliotece uzmysłowiłem sobie również, że nie będzie takie łatwe sprawdzanie poprawności numeru telefonu. W każdym kraju jest on inny i dlatego będzie pradopodobnie trzeba pytać dodatkowo usera o prefix międzynarodowy numeru. Ale to nie problem - zostanie to dorobione. Metoda logowania natomiast ma za zadanie sprawdzenia, czy wpisaliśmy prawidłowy kod autoryzujący. To już będzie później po stronie logiki serwerowej. W tym momencie jedynie sprawdzamy, czy wprowadzono liczbę większą niż 0 - jeśli tak to otwieramy główne okno aplikacji.

Ten ostatni krok przysporzył mi niemałych problemów ale okazało się ostatecznie, że udało mi się dojść do celu z sukcesem. W electronie istnieją dwa procesy. Main i Renderer - główny i renderujący. Pierwszy z nich odpowiada za obsługę całej aplikacji oraz dostępy do maszyny lokalnej. Drugi proces odpowiedzialny jest za renderowanie treści - czyli to co robimy. Okazało się, że proces renderujacy nie może otworzyć nowego okna. Ale może to zrobić proces główny. Jak zatem to zrobić? Nalezy użyć IPC. IPC pozwala na asynchroniczną komunikację pomiędzy procesem głównym i renderującym. Jak widzicie tutaj wysłaliśmy notyfikację do do głównego procesu nazwie 'open-main-window'. Dlatego w pliku index.js w głównym katalogu naszej aplikacji musiałem dodać taki kod:

```js
ipcMain.on('open-main-window', function() {
    mainScreen = new BrowserWindow({
        center: true,
        maximizable: true,
        title: 'MassCo'
    });

    mainScreen.on('closed', () => {
        mainScreen = null;
    });

    mainScreen.on('show', () => {

    });

    mainScreen.loadURL('file://' + __dirname + '/app/index.html');

    startScreen.close();
});
```

Dzięk itemu możemy otworzyć główne okno aplikacji i po jego uruchomieniu zamknąć okno startowe.

### CSS ###
O samym CSS nie bedę opowiadałza wiele. Większych też problemów nie miałem, bo jednak generalnie sam HTML i CSS to wiele osób umie. W mniejszym lub większym stopniu. Myślę, ze osobiscie umiem go dość w znaczny mstopniu aczkolwiek przy okazji tej aplikacji chętnie douczę się animacji i flex'ów. Dlatego nie będę Wam wrzucać już tutaj pliku CSS. Zapraszam na gita do obejrzenia zmian w repozytorium i zobaczenia jak to wygląda.

Dopowiem też, ze póki co korzystam z plików css. Ale docelowo w głównej już aplikacji chcę wykorzystać LESS. Stwierdziłem, ze do samego ekranu logowania nie ma to większego sensu, zwłąszcza, ze ekran jest prosty. 

## Podsumowanie ##

Jak już dwukrotnie wspomniałem zapraszam Was do przeglądania zmian w [repozytorium][2]. Jest tam o wiele więcej niż pokazuję to tutaj. 

Dodatkowo zapraszam na [blog mego kolegi djfoxer][3], który chce zadbać w konkursie o nasze zdrowie i produktywność w trakcie pracy dzięki wtyczce do VisualStudio! 


  [1]: https://github.com/duszekmestre/MassCo/commit/00f93f44f1a2f4552228c9dabf088e923ea639bd
  [2]: https://github.com/duszekmestre/MassCo
  [3]: https://www.dobreprogramy.pl/djfoxer
  [4]: {{ site.baseurl }}/images/login-screen.PNG