---
layout:     post
title:      Token based authentication - cz. 2
date:       2017-03-18 22:00
summary:    Dzisiaj znów troszkę kodowania. Pierwsze kroki ku przygotowaniom do logowania i rejestracji w ASP.NET Core Web API. Zapraszam.
tags:       DSP2017 token asp.net authentication
categories: dsp2017
---

Dzisiaj znów troszkę kodowania. Pierwsze kroki ku przygotowaniom do logowania i rejestracji w ASP.NET Core Web API. Zapraszam.

## Tworzenie projektu ##

Nie ma co długo się zastanawiać. Trzeba zacząć kodować :) Pierwsze kroki zaczynamy od utworzenia projektu ASP.NET Core. Można to zrobić dwojako. Albo użyć node.js i zaincjalizować projekt za jego pomocą. Można też użyć Visual Studio. Pomyślałem, ze przy okazji projektu wykorzystam najnowszą wersję VS 2017. I za jego pomocą utworzę projekt.

### Nowy projekt ###

Przedstawię to w obrazkach:

![New project][1]

![Select Web API][2]

Po trych dwóch krokach jesteśmy prawie gotowi. Aby wykorzystać istniejącą implementację token based authenitcation musimy dodać jedną paczkę do projektu:

![Add JWTBearer nuget package][3]

Oczywiście możemy to też zrobić za pomocą polecenia w okienku Package Manager Console:

```
Install-Package Microsoft.AspNetCore.Authentication.JwtBearer
```

Jesteśmy gotowi do kolejnego kroku:

### Implementacja Token Based Authentication ###

Mnie w projekcie interesuje Bearer authentication, czy token przesyłany bedzie za pomocą nagłówków w żądaniu HTTP. Pierwszy krokiem będzie dodanie nowych ustawień dla naszego "mechanizmu" w pliku **appsettings.json**:

```json
{
    "Logging": {
        "IncludeScopes": false,
        "LogLevel": {
            "Default": "Warning"
        }
    },
    "TokenAuthentication": {
        "SecretKey": "secretkey_secretkey123!",
        "Issuer": "MassCoIssuer",
        "Audience": "MassCoAudience",
        "TokenPath": "/api/token"
    }
}
```

Do naszego projektu dodajemy partial klasy Startup w pliku **Startup.Auth.cs** (nie zapominajmy o oznaczeniu jej jako *partial* zarówno w tym pliku jak i w pliku **Startup.cs**):

```csharp
public partial class Startup
{
    public SymmetricSecurityKey signingKey;

    private void ConfigureAuth(IApplicationBuilder app)
    {
        var signingKey = new SymmetricSecurityKey(Encoding.ASCII.GetBytes(Configuration.GetSection("TokenAuthentication:SecretKey").Value));

        var tokenProviderOptions = new TokenProviderOptions
        {
            Path = Configuration.GetSection("TokenAuthentication:TokenPath").Value,
            Audience = Configuration.GetSection("TokenAuthentication:Audience").Value,
            Issuer = Configuration.GetSection("TokenAuthentication:Issuer").Value,
            SigningCredentials = new SigningCredentials(signingKey, SecurityAlgorithms.HmacSha256),
            IdentityResolver = GetIdentity
        };

        var tokenValidationParameters = new TokenValidationParameters
        {
            // The signing key must match!
            ValidateIssuerSigningKey = true,
            IssuerSigningKey = signingKey,
            // Validate the JWT Issuer (iss) claim
            ValidateIssuer = true,
            ValidIssuer = Configuration.GetSection("TokenAuthentication:Issuer").Value,
            // Validate the JWT Audience (aud) claim
            ValidateAudience = true,
            ValidAudience = Configuration.GetSection("TokenAuthentication:Audience").Value,
            // Validate the token expiry
            ValidateLifetime = true,
            // If you want to allow a certain amount of clock drift, set that here:
            ClockSkew = TimeSpan.Zero
        };


        app.UseJwtBearerAuthentication(new JwtBearerOptions
        {
            AutomaticAuthenticate = true,
            AutomaticChallenge = true,
            TokenValidationParameters = tokenValidationParameters
        });

        app.UseMiddleware<TokenProviderMiddleware>(Options.Create(tokenProviderOptions));
    }

    private Task<ClaimsIdentity> GetIdentity(string username, string password)
    {
        // DEMO CODE, DON NOT USE IN PRODUCTION!!!
        if (username == "TEST" && password == "TEST123")
        {
            return Task.FromResult(new ClaimsIdentity(new GenericIdentity(username, "Token"), new Claim[] { }));
        }

        // Account doesn't exists
        return Task.FromResult<ClaimsIdentity>(null);
    }
}
```

No i wiem. Co tu się dzieje? :O Także po kolei:

Metoda ConfigureAuth jest potrzebna nam do skonfigurowania autentykacji. Początkowo pobieramy SecretKey z naszych ustawień. Powinien on być tajny. W ramach tego bloga wrzucać będę do repozytorium dane byle jakie żebyście mogli ewentualnie ładnie potestować. Następnie tworzymy opcje wykorzystujac dane z konfiguracji. Na sam koniec wykorzystujemy gotowy mechanizm z paczki, którą dodaliśmy na początku. Ostatnia linijka jest znacząca. Określamy tutaj, że będziemy wykorzystywać swoją implementację tokenów. A oto i ona:

```csharp
public class TokenProviderMiddleware
{
    private readonly RequestDelegate _next;
    private readonly TokenProviderOptions _options;
    private readonly JsonSerializerSettings _serializerSettings;

    public TokenProviderMiddleware(
        RequestDelegate next,
        IOptions<TokenProviderOptions> options)
    {
        _next = next;

        _options = options.Value;
        ThrowIfInvalidOptions(_options);

        _serializerSettings = new JsonSerializerSettings
        {
            Formatting = Formatting.Indented
        };
    }

    public Task Invoke(HttpContext context)
    {
        // If the request path doesn't match, skip
        if (!context.Request.Path.Equals(_options.Path, StringComparison.Ordinal))
        {
            return _next(context);
        }

        // Request must be POST with Content-Type: application/x-www-form-urlencoded
        if (!context.Request.Method.Equals("POST")
           || !context.Request.HasFormContentType)
        {
            context.Response.StatusCode = 400;
            return context.Response.WriteAsync("Bad request.");
        }


        return GenerateToken(context);
    }

    private async Task GenerateToken(HttpContext context)
    {
        var username = context.Request.Form["username"];
        var password = context.Request.Form["password"];

        var identity = await _options.IdentityResolver(username, password);
        if (identity == null)
        {
            context.Response.StatusCode = 400;
            await context.Response.WriteAsync("Invalid username or password.");
            return;
        }

        var now = DateTime.UtcNow;

        // Specifically add the jti (nonce), iat (issued timestamp), and sub (subject/user) claims.
        // You can add other claims here, if you want:
        var claims = new Claim[]
        {
            new Claim(JwtRegisteredClaimNames.Sub, username),
            new Claim(JwtRegisteredClaimNames.Jti, await _options.NonceGenerator()),
            new Claim(JwtRegisteredClaimNames.Iat, new DateTimeOffset(now).ToUniversalTime().ToUnixTimeSeconds().ToString(), ClaimValueTypes.Integer64)
        };

        // Create the JWT and write it to a string
        var jwt = new JwtSecurityToken(
            issuer: _options.Issuer,
            audience: _options.Audience,
            claims: claims,
            notBefore: now,
            expires: now.Add(_options.Expiration),
            signingCredentials: _options.SigningCredentials);
        var encodedJwt = new JwtSecurityTokenHandler().WriteToken(jwt);

        var response = new
        {
            access_token = encodedJwt,
            expires_in = (int)_options.Expiration.TotalSeconds
        };

        // Serialize and return the response
        context.Response.ContentType = "application/json";
        await context.Response.WriteAsync(JsonConvert.SerializeObject(response, _serializerSettings));
    }

    private static void ThrowIfInvalidOptions(TokenProviderOptions options)
    {
        if (string.IsNullOrEmpty(options.Path))
        {
            throw new ArgumentNullException(nameof(TokenProviderOptions.Path));
        }

        if (string.IsNullOrEmpty(options.Issuer))
        {
            throw new ArgumentNullException(nameof(TokenProviderOptions.Issuer));
        }

        if (string.IsNullOrEmpty(options.Audience))
        {
            throw new ArgumentNullException(nameof(TokenProviderOptions.Audience));
        }

        if (options.Expiration == TimeSpan.Zero)
        {
            throw new ArgumentException("Must be a non-zero TimeSpan.", nameof(TokenProviderOptions.Expiration));
        }

        if (options.IdentityResolver == null)
        {
            throw new ArgumentNullException(nameof(TokenProviderOptions.IdentityResolver));
        }

        if (options.SigningCredentials == null)
        {
            throw new ArgumentNullException(nameof(TokenProviderOptions.SigningCredentials));
        }

        if (options.NonceGenerator == null)
        {
            throw new ArgumentNullException(nameof(TokenProviderOptions.NonceGenerator));
        }
    }
}
```

Ponownie dużo się dzieje. Tak to jest jak robi się coś dedykowanego. W konstruktorze ładujemy ustawienia i je walidujemy. W metodzie Invoke sprawdzamy, czy request, który przyszedł nas dotyczy (sprawdzamy ścieżkę). Jeśli nie to puszczamy request dalej. Następnie walidujemy, czy był on przesłany metodą POST. Jako odpowiedź generujemy token. Metoda GenerateToken zajmuje sie pobraniem danych, walidacją oraz generowaniem tokenu.

>Czym jest Middleware? Generalnie ASP.NET Core opiera się na przetwarzaniu zapytań. I jeśli chcemy aby w całe flow wpiąć jakiś krok to musimy posiadać tzw Middleware, które jest takim właśnie krokiem przetwarzania requestu. I ten krok moze albo puścić request do następnego kroku albo odrazu zwrócić dane. Dzięki temu można łatwo budować ścieżkę, jaką przejdzie każdy request wysłany do aplikacji. 

To jeszcze tylko jeden plik został (z tych customowych). TokenOptions:

```csharp
public class TokenProviderOptions
{
    public string Path { get; set; } = "/token";

    public string Issuer { get; set; }

    public string Audience { get; set; }

    public TimeSpan Expiration { get; set; } = TimeSpan.FromMinutes(30);

    public SigningCredentials SigningCredentials { get; set; }

    public Func<string, string, Task<ClaimsIdentity>> IdentityResolver { get; set; }

    public Func<Task<string>> NonceGenerator { get; set; }
        = () => Task.FromResult(Guid.NewGuid().ToString());
}
```

Tutaj krótko i opisywać nawet nie będę. Ale za to wrócę do pliku **Startup.cs**. W nim na samym końcu pojawiła się metoda *GetIdentity*. W niej odbywa się najważniejszy krok autentykujący. Sprawdzmy tam, czy przesłane dane do logowania są prawdziwe. W tym momencie jak sami widzicie to wygląda bardziej jak mockup. Głównie potrzebny do przetestowania. A propos...

### Testujemy ###

Po utworzeniu projektu mamy domyślnie zaimplementowany ValueController. Aby zweryfikować, czy nasz mechanizm działa musimy poczynić jeszcze dwa kroki.

1. Dodajemy konfigurację do metody Configure w **Startup.cs**:

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
{
    loggerFactory.AddConsole(Configuration.GetSection("Logging"));
    loggerFactory.AddDebug();

    ConfigureAuth(app);

    app.UseMvc();
}
```

Bardzo ważna kwestia:

**ConfigureAuth musi pojawić się przed UseMvc.**

Dlaczego? kolejność Middleware jest ważna. I gdyby UseMvc pojawiło się pierwsze - wtedy akcje MVC wpięły by się w routing i widzac atrybut **Authorize** przejęłoby kontrolę nad jego obsługa i jako odpowiedź dostamieny *401 Unauthorized*.

2. Nad metodą ValueController dodajemy atrybut [Authorize]

```csharp
[Authorize]
public class ValuesController : Controller
```

Teraz tylko build, run, i ...

.

.

.

Ano i. Pusty ekran. Działa, czy nie? Najlepiej będzie to zrobić inaczej. Pozostawmy tą aplikację uruchomioną i przejdźmy do PostMana (lub innego super narzędzia do tworzenia requestów). I wywołajmy akcję (POST):

```HTTP
POST api/token
Content-Type: application/x-www-form-urlencoded
username=TEST&password=TEST123
```

Jako odpowiedź otrzymamy JSONa o podobnej treści:

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJURVNUIiwianRpIjoiMjk2ZTUxMmUtMDgwMy00YTI1LWJmNTYtZWZhZGE2NDk3MTJmIiwiaWF0IjoxNDg5ODY5NTUyLCJuYmYiOjE0ODk4Njk1NTIsImV4cCI6MTQ4OTg3MTM1MiwiaXNzIjoiTWFzc0NvSXNzdWVyIiwiYXVkIjoiTWFzc0NvQXVkaWVuY2UifQ.Yrl_DDLb2QXRQTj472nq7YCjuAzP3zuvaslEa4DTZ58",
  "expires_in": 1800
}
```

teraz znając token wywołujemy akcję GET do naszego zasobu:

```HTTP
GET /api/values HTTP/1.1
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJURVNUIiwianRpIjoiYTEzMTAxN2QtMWZlNS00N2EyLWI5ZjEtZGRhMTZlZWQ4NDk1IiwiaWF0IjoxNDg5ODY5NjY2LCJuYmYiOjE0ODk4Njk2NjYsImV4cCI6MTQ4OTg3MTQ2NiwiaXNzIjoiTWFzc0NvSXNzdWVyIiwiYXVkIjoiTWFzc0NvQXVkaWVuY2UifQ.MJFaghm3jZDJRX7tjFh8m0nq99QZTxHqm2FIovwXg-g
```

I jako piękną odpowiedź otrzymujemy dane:

```json
[
  "value1",
  "value2"
]
```

No i pięknie! :) A teraz wystarczy wpróbować jak zadziała, gdy nie prześlemy tokenu:

```HTTP
GET /api/values HTTP/1.1
```

Odpowiedź: **401 Unauthorized**

No i ostatni test. Zły token:

```HTTP
GET /api/values HTTP/1.1
Authorization: Bearer heheszki
```

Odpowiedź: **401 Unauthorized**

Działa przepięknie.

## Podsumowanie ##

Jak widzimy. Troszkę kodu było trzeba. Następnym razem postaram się dodać repozytorium InMemory (potestujemy nową funkcjonalność EF Core). A po repozytorium Logowanie i Rejestracja użytkownika! Do usłyszenia!

P.S. Przypominam o zajrzeniu na kilka ciekawych stron:

1. [Developing token authentication using ASP.NET Core][4] - w swoim wpisie bardzo dużo bazowałem na tym artykule
2. [Strona projektu MassCo][5]
3. [Projekt ASP.NET Core][6]
4. [Mój funpage na Facebooku][7]


  [1]: {{ site.baseurl }}/images/core-new-project.PNG
  [2]: {{ site.baseurl }}/images/core-web-api.PNG
  [3]: {{ site.baseurl }}/images/core-nuget.PNG
  [4]: https://dev.to/samueleresca/developing-token-authentication-using-aspnet-core
  [5]: https://github.com/duszekmestre/MassCo
  [6]: https://github.com/aspnet
  [7]: https://www.facebook.com/duszekarciszewski/