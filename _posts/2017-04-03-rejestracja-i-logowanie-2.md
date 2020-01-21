---
layout:     post
title:      Rejestracja i logowanie - cz.2, SMS
date:       2017-04-03 17:46
summary:    Logowanie kodem z smsa. Zróbmy to łatwo i za darmo!
tags:       DSP2017 authentication .NET Core API SMS

---

Logowanie kodem z smsa. Zróbmy to łatwo i za darmo!

## Darmowe SMSy ##

Tak jak wspominałem - postanowiłem pójść z logowaniem w innym kierunku. Nie chcę męczyć użytkownika hasłami. Po co ma to pamietać. Jedyne co wystarczy aby pamiętał, to swój numer telefonu. Dzięki automatycznie wyślę do niego SMSem kod logowania i dostęp do konta jest natychmiastowy. Hm. I zapewne pomyślicie, że skoro tak to wystarcy dostęp do telefonu i po sprawie. No w sumie racja. Ale z drugiej strony - myślę, że coraz więcej osób ma też hasło do telefonu, prawda? :)

No ale te SMSy. Przejrzałem kilka różnych rozwiązań do wysyłki smsów. Najbardziej przypadł mi do gustu serwis [Twilio][1]. Ma jednak jedną zasadniczą wadę. Nie jest darmowy. Tak jak każdy, który znalazłem serwis tego typu. Bo w sumie kto by rozdawał smsy za darmo? :P W związku z czym poszukiwałem rozwiązania darmowego. I znalazłem! Co prawda takie rozwiązanie na produkcji nie przejdzie ale w tym momencie robię development. A do produkcji z projektem jeszcze kawałek. Dlatego z pomocą przyszedł [SMS Gateway][2]. Super mega proste rozwiązanie. Instalujemy aplikację na naszym telefonie, logujemy się i już mamy bramkę SMS za darmo! No to teraz przyszedł czas na requesty :)

### Integracja z SMS API ###

Uwielbiam IoC dlatego napisałem najpierw prosty interfejs:

```csharp
public interface ISmsSender
{
    Task<bool> SendSmsAsync(string number, string message);
}
```

W prostocie siła. Skoro mamy już interfejs to już czas na implementację:

```csharp
public async Task<bool> SendSmsAsync(string number, string message)
{

    using (var client = new HttpClient())
    {

        var content = new FormUrlEncodedContent(new[]
        {
            new KeyValuePair<string, string>("email", smsOptions.Username),
            new KeyValuePair<string, string>("password", smsOptions.Password),
            new KeyValuePair<string, string>("device", smsOptions.DeviceID),
            new KeyValuePair<string, string>("number", number),
            new KeyValuePair<string, string>("message", message)
        });

        var response = await client.PostAsync(smsGatewayUrl, content);

        if (response.IsSuccessStatusCode)
        {
            return true;
        }

        return false;
    }
}
```

W powyższej metodzie zgodnie z API wysyłam kilka parametrów:

1. **email** - zarejestrowanego użytkownika w API
2. **password** - hasło podane przy rejestracji
3. **device** - identyfikator urządzenia, który widzimy w aplikacji mobilnej po zalogowaniu się
4. **number** - numer adresata
5. **message** - wiadomość do wysłania
 
Zero wysiłku. Teraz tylko POST na adres [http://smsgateway.me/api/v3/messages/send][3] i poszło. Telefon natychmiast zauważył nową wiadomość i ją wysłał. Chwila potem - bzy, bzy - sms. :) Proste i łatwe. 

### DI - tegister & resolve ###

Oczywiscie tak napisaną implementację rejestrujemy w kontenerze:

```csharp
services.AddTransient<ISmsSender, SmsSender>();
```

Powyższa rejestracja powoduje, ze za każdym requestem będzie utworzona nowa instancja obiektu. A jak do niej się teraz dostać? Wystarczy do konstruktora naszego kontrolera dorzucić:

```csharp
private ISmsSender SmsSender { get; set; }

private IUnitOfWork UnitOfWork { get; set; }
private IDistributedCache Cache { get; set; }
private ICodeGenerator CodeGenerator { get; set; }

public AccountController(
    IUnitOfWork unitOfWork,
    IDistributedCache cache,
    ISmsSender smsSender,
    ICodeGenerator codeGenerator)
{
    this.UnitOfWork = unitOfWork;
    this.Cache = cache;
    this.SmsSender = smsSender;
    this.CodeGenerator = codeGenerator;
}
```

I dzięki temu w metodach mamy już dostęp.

### Code generator ###

W powyższym kawałku kodu zauważyć też można, że dodałem ICodeGenerator. Zrobiłem to na takiej samej zasadzie jak ISmsSender. Przygotowałem też implementację, która polega na losowaniu liczby 4 cyfrowej. Chciałem aby mieć jedną implementację generowania kodu jednorazowego w całej aplikacji. Dodatkowo zarejestrowałem ją jako singleton:

```csharp
services.AddSingleton<ICodeGenerator, NumericCodeGenerator>();
```

Zrobiłem to po to, aby klasa Random istniała tylko jedna dla generatora. To daje mniejszą przewidywalność podczas kolejnych losowań liczby.

### Secret values ###

Kilka powyższych danych takich jak email, password, deviceID są dość poufne i nie chciałbym aby ktoś użył moich danych. Jako iż pisze się oprogramowanie OpenSource to wszystko jest dostępne. Dlatego aby tego uniknąć dodałem dodatkowy plik konfiguracyjny: **appsettings.secret.json**. Następnie w klasie Startup w konstruktorze usupełniłem ten wpis:

```csharp
public Startup(IHostingEnvironment env)
{
    var builder = new ConfigurationBuilder()
        .SetBasePath(env.ContentRootPath)
        .AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
        .AddJsonFile("appsettings.secret.json", optional: false, reloadOnChange: true)
        .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true)
        .AddEnvironmentVariables();
    Configuration = builder.Build();
}
```

i dodałem ten plik do **.gitignore**. Dzieki takim zabiegom nie uda się uruchomić aplikacji bez tego pliku + nie wpadnie on do repozytorium. Jest jeszcze możliwość przechowywania haseł w Secret Manager. Więcej informacji możecie uzyskać pod adresem: [https://docs.microsoft.com/en-us/aspnet/core/security/app-secrets][4]. JA z tej opcji nie skorzystałem, ponieważ jest ona dostępna jedynie przy użyciu toola *dotnet*. Ja natomiast korzystając z VisualStudio nie mam do niego dostępu.

## Ciekawe linki ##

Oto spis kilku ciekawych linków związanych (i nie) z tematem:

1. [Dependecy Injection w ASP. NET Core][5]
2. [Wysyłka SMS do konkretnego numeru][6]
3. Oczywiście [moje repozytorium][7] :)
4. [Blog djfoxera o dodatku do VS o produktywności][8]

## Co dalej? ##

W temacie logowania i rejestracji pozostał mi jeuż tylko front-end. I bedzie to ostatnia część w serii. Po drodze pojawi się jeszcze krótki ale treściwy wpis o konfigurowaniu w ASP.NET Core. Do usłyszenia!


  [1]: https://www.twilio.com/
  [2]: https://smsgateway.me/
  [3]: http://smsgateway.me/api/v3/messages/send
  [4]: https://docs.microsoft.com/en-us/aspnet/core/security/app-secrets
  [5]: https://docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection
  [6]: https://smsgateway.me/sms-api-documentation/messages/send-message-to-number
  [7]: https://github.com/duszekmestre/MassCo
  [8]: https://www.dobreprogramy.pl/djfoxer