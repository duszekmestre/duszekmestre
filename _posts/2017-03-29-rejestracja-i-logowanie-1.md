---
layout:     post
title:      Rejestracja i logowanie - cz.1, API
date:       2017-03-29 19:16
summary:    To pierwsza część podwójnego wpisu, który będzie opowiadał o logowaniu oraz rejestracji. W tym wpisie opiszę część implementacji po stronie API.
tags:       DSP2017 authentication .NET Core API Redis cache

---

To pierwsza część podwójnego wpisu, który będzie opowiadał o logowaniu oraz rejestracji. W tym wpisie opiszę część implementacji po stronie API.

### AccountController ###

Tutaj wydarzyło się najwięcej. Generalnie przyszedłczas aby zaimplementować logowanie wraz z rejestracją. To co prawda jest dopiero pierwszy krok, ponieważ zrobię to na razie po stronie API. Wyszczególniłem kilka metod związanych z kontem:

1. [x] GET  /api/account/me
2. [x] GET  /api/account/{id}
3. [ ] POST /api/account/validate
4. [ ] POST /api/account
5. [ ] PUT  /api/account/confirm

Zaznaczone akcje wymagają autoryzacji. Ale po kolei. Aby zalogować się trzeba wywołać akcję **POST /api/account/validate**. Do rejestracji natomiast wymagane są w kolejności **POST /api/account** oraz **PUT /api/account/confirm**. Dlaczego tak? Jeśli pamiętacie ekran logowania to pierwszym krokiem było podanie numeru telefonu do zalogowania. W tym momencie będzie trzeba wysłać metodę validate, która zweryfikuje, czy podane konto istnieje (lub czy numer został wpisany prawidłowo). Jako odpowiedź moze zwrócić nam błąd w formacie numeru, brak konta, lub AccountID. 

W przypadku, gdy jest brak konta pojawi się w formularzu pole do wprowadzania nazwy użytkownika. Wtedy należy wykonać akcję utworzenia konta. A jako ostatni krok rejestracji wywołujemy confirm. Ten ostatni krok ma za zadanie zwalidować wysłany SMSem kod. Póki co implementacji wysyłki SMS nie robiłem.

Natomiast jest przypadek, gdy konto istnieje. Wtedy automatycznie zostanie wysłany kod SMSem na podany numer i użytkownik będzie musiał podać ten kod na formularzu i przesłać do metody generującej token. Dodatkowo kod ten bedzie ważny jedynie 15 min.

### Cache ###

To przechowywania kodu użyłem wbudowanej w ASP.NET Core implementacji [DistributedCache][1]. Jest to cache, który jest współdzielony pomiędzy wszystkeie implementacje aplikacji. A jak to jest zrobione? Są dwie wpudowane opcje:

1. SQL
2. Redis Cache

Ze względu na to iż miałem z oboma rozwiązaniami do czynienia wybrałem Redis ze względu na szybkość działania. Jest to dedykowany serwer cache. [Więcej o redisie możecie poczytać na ich stronie][2].

W celu skonfigurowania cache redisa wystarczy w klasie *Startup* w metodzie *ConfigureServices* dodać taki wpis:

```csharp
services.AddDistributedRedisCache(options =>
{
    options.Configuration = "localhost";
    options.InstanceName = "MassCo";
});
```

Następnie jeśli chcemu użyć cache powinniśmy użyć za pomocą Dependency Injection *IDistributedCache* (przekazać jako parametr do konstruktora w controllerze). I zapis do cache wygląda następująco:

```csharp
await Cache.SetStringAsync(userAccount.Id.ToString(), confirmationCode);
```

a odczyt:

```csharp
var code = await Cache.GetStringAsync(request.AccountId.ToString());
```

Proste? Proste. I działa? NIE! Dlaczego? Bo jeszcze jedna ważna rzecz. Najlepiej za pomocą [chocolatey][3] zainstalować serwis redisa na Windows za pomocą komendy:

```
choco install redis-64
```

Po zainstalowaniu uruchamiamy serwer komendą:

```
redis-server
```

No i teraz działa! :)

### TokenMiddleware - IdentityResolver ###

Tutaj nastąpiły małe zmiany. Pierwotnie w naszej implementacji był username i password. Jak już zauważyliscie zapewne powiedziałem, ze teraz wymagany jest kod i AccountID. W związku z czym małe zmiany:

```csharp
rivate async Task GenerateToken(HttpContext context)
{
    var accountIdForm = context.Request.Form["accountid"];
    var confirmationCode = context.Request.Form["confirmationcode"];

    if (!int.TryParse(accountIdForm, out var accountId))
    {
        await WrongAcccountOrConfirmationCode(context);
        return;
    }

    var identity = await _options.IdentityResolver(context, new AuthenticateVM
    {
        AccountId = accountId,
        ConfirmationCode = confirmationCode
    });
    
    ...
}
```

Czyli pobieramy z formularza accountid oraz confirmationcode. A samo sprawdzenie polega w tym momencie na:

```csharp
private async Task<ClaimsIdentity> GetIdentityAsync(HttpContext context, AuthenticateVM request)
{
    var unitOfWork = serviceProvider.GetService<IUnitOfWork>();

    var userAccount = await unitOfWork.AccountRepository.GetAsync(request.AccountId);

    if (userAccount == null || !userAccount.IsConfirmed)
    {
        return null;
    }

    var cache = serviceProvider.GetService<IDistributedCache>();
    var code = await cache.GetStringAsync(userAccount.Id.ToString());

    if (request.ConfirmationCode == code)
    {
        return new ClaimsIdentity(new GenericIdentity(userAccount.Id.ToString(), "Token"), new Claim[] { });
    }

    // Account doesn't exists
    return null;
        }
```

Widzimy tutaj, że wykorzytałem pobieranie implementacji IUnitOfWork z kontenera IoC. serviceProvider został przekazany do tej klasy jako parametr z metody *Configure* w klasie *Startup*. Do tej metody trzeba jako parametr przekazać to czego potrzebujemy jeśłi chcemy aby zadziałało DependencyInjection. W naszym wypadku to: 

```csharp
IServiceProvider serviceProvider
```

### Account ID ###

To kolejna dośc istotna rzecz. Korzystamy w naszym wypadku z ClaimIdentity. Niestety ono nie jest takie bezpośrednie w związku z czym jeśli chcemy dowiedzieć się jakie jest ID aktualnie zalogowanego użytkownika musimy dopisać kilka zmian. W klasie *TokenProviderMiddleware* w metodzie *Invoke* w miejscu generowania tablicy claims dodałem dodatkowy wpis:

```csharp
var claims = new Claim[]
{
    new Claim(IDENTITY_KEY, accountId.ToString()),
    new Claim(JwtRegisteredClaimNames.Sub, accountId.ToString()),
      new Claim(JwtRegisteredClaimNames.Jti, await _options.NonceGenerator()),
      new Claim(JwtRegisteredClaimNames.Iat, new DateTimeOffset(now).ToUniversalTime().ToUnixTimeSeconds().ToString(), ClaimValueTypes.Integer64)
};
```

Jest tutaj pewna stała *IDENTITY_KEY*, której wartość to: *"Account:Id"*. W takiego claima wpisuję ID konta aktualnie zalogowanego użytkownika. Dodatkowo dodałem abstrakcyjną klasę dla wszystkich Controllerów:

```csharp
public abstract class BaseController : Controller
{
    public int UserId
    {
        get
        {
            var accountIdString = (this.User.Identity as ClaimsIdentity)?.Claims.FirstOrDefault(x => x.Type == TokenProviderMiddleware.IDENTITY_KEY)?.Value;

            int.TryParse(accountIdString, out int accountId);

            return accountId;
        }
    }
}
```

Dzięki temu każdy controller, który podziedziczy po tej klasie będzie miał dostęp do ID aktualnie zalogowanego użytkownika i będzie można wtedy wykonywać akcje w danym kontekście. To bardzo ważne w niektórzych wypadkach.

### Co jeszcze? ###

Troszkę jeszcze zmian poczyniłem. Dodałem AccountRepository, które dziedziczy po generycznym repozytorium i wrzuciłem je do UnitOfWork. Dzięk itemu mogę prócz istniejących metod dopisać też własne indywidualne metody do obsługi kont. 

Miałem jeszcze sporo problemów z migracjami EF. Nadal w sumie mam. Rozwiązania nie znalazłem. Mimo iż migracja się wykonała to zakończyła się błędem. W bazie zmiany się naniosły a błąd i tak wyświetla :) Życie. To tylko Microsoft :P 

Po szczegóły wszystkich zmian zapraszam do przejrzenia [commita][4].

W kolejnym wpisie w cyklu postaram się już w naszej aplikacji połączyć z API i stworzyć serwis do łatwego zarządzania tokenem. A w tym tygodniu opiszę jeszcze jak dodać automatyczną dokumentację za pomocą [Swaggera][5] do naszego API.


  [1]: https://docs.microsoft.com/en-us/aspnet/core/performance/caching/distributed
  [2]: https://redis.io/
  [3]: https://chocolatey.org/
  [4]: https://github.com/duszekmestre/MassCo/commit/b15b9faa7546e9cc9c89c9349025dd32475f3cb4
  [5]: http://swagger.io/