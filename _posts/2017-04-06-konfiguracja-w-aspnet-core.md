---
layout:     post
title:      Konfiguracja w ASP.NET Core
date:       2017-04-06 18:32
summary:    Gdy odkryłem możliwości, które daje mi konfiguracja pojawił się uśmiech na mojej twarzy. Uśmiech postępu. Dlaczego? Przekonajcie się sami.
tags:       DSP2017 .NET Core Configuration
categories: dsp2017
---

Gdy odkryłem możliwości, które daje mi konfiguracja pojawił się uśmiech na mojej twarzy. Uśmiech postępu. Dlaczego? Przekonajcie się sami.

## Configuration API ##

Tutaj nie chodzi o to, że użytkowanie konfiguracji jest mega hiper wypasione ale o to, że posiada niemalże wszystko to, czego dotej pory nie miała. Poniżej będę opisywać chyba same zalety, bo podczas dotychczasowego programowania na wady tego rozwiązania się nie natknąłem.

**UWAGA**

Wiele z przykładów zostało zaczerpniętych bezpośrednio ze strony: [https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration][1], na którą serdecznie zapraszam.

### Dostawcy konfiguracji ###

Po pierwsze - nie jesteśmy poniekąd zmuszeni do tego gdzie i jak przechowujemy konfigurację. Mamy większą wolność i decyzyjność w tym temacie. Z dostarczonych przez zespół ASP.NET Core dostawców można wybrać dokładnie to co nam pasuje:

1. Pliki (*.ini, *.json, *.xml)
2. Argumenty wiersza poleceń
3. Zmienne środowiskowe
4. Pamięć
5. Szyfrowany magazyn użytkownika
6. [Azure Key Vault][2]

Ale to nie wszystko. Bo tą listę możemy rozszerzać sami doinstalowując paczkami lub pisząc swój własny magazyn!

### Hierarchiczność ###

Cóż tu dużo mówić. Sami zobaczcie na przykład:

```json
{
    "ConnectionStrings": {
        "MassCoLocalDB": "Server=(localdb)\\MSSQLLocalDB;Initial Catalog=MassCo.Database;Integrated Security=True;MultipleActiveResultSets=true"
    },
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
    },
    "Settings": {
        "UseInMemoryDatabase": false
    },
    "SmsConfiguration": {
        "Username": "",
        "Password": "",
        "DeviceID":  ""
    }                            
}

```

Wszystko ma swój porządek. Dzielimy to dokładnie tak jak uważamy, zbieramy w obiekty i potem możemy uzyskać dostęp do poszczególnych kawałków.

### Get started ###

Aby zacząć pracę z konfiguracją należy jak najbliżej początku naszej aplikacji zainicjalizować ją. W jaki sposób? Np tak:

```csharp
var builder = new ConfigurationBuilder()
    .SetBasePath(env.ContentRootPath)
    .AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
    .AddJsonFile("appsettings.secret.json", optional: false, reloadOnChange: true)
    .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true)
    .AddEnvironmentVariables();
Configuration = builder.Build();
```

Używamy do tego klasy [**ConfigurationBuilder**][3]. Używamy ją zgodnie z konwencją chain invoke (lub nie :P).

1. **SetBasePath** - Ustawiamy ścieżkę root, w której posiadamy konfigurację
2. **AddJsonFile** - Dodajemy plik JSON jako naszą konfigurację (lub jakiś inny jeśli tak zdecydujemy). Pierwsze wywołanie zazwyczaj nie będzie opcjonalne. Klejne wywołania już zależnie opcjonalne mogą ale też nie muszą być. Dzięki takiemu zabiegowi możemy kolejnymi plikami nadpisywać lub rozszerzać konfigurację. Czyli dokonamy **transformacji** konfiguracji. I to jest MEGA. Bo nareszcie w każdym rodzaju projektu .NET Core mamy taką możliwość i to z paczki - bez dodatków.
3. **AddEnvironmentVariables** - sama nazwa wskazuje. Dodajemy Zmienne środowiskowe.
 
Na sam koniec już tylko **Build()** i konfiguracja gotowa. A jak z niej korzystać?

### Używanie konfiguracji ###

Jest kilka sposobów. Oto i one:

#### Direct access ####

Moim zdaniem najmniej polecana metoda ale często konieczna. Generalnie odwołujemy się bezpośrednio do wartości konkretnego wpisu poprzez odwołanie do spłaszczonego słownika, np:

1. Configuration["option1"]
2. Configuration["subsection:suboption1"]
3. Configuration["wizards:0:Age"] - odwołanie do indeksu tablicy

Sami widzicie - dość pracochłonne i niewygodne. Literówka i... BANG.

Ale druga opcja jest już odrobinę lepsza. Nadal literówki są problemem ale już typpy nie. Mając taką konfigurację:

```json
{
  "AppConfiguration": {
    "MainWindow": {
      "Height": "400",
      "Width": "600",
      "Top": "5",
      "Left": "11"
    }
  }
}
```

Możemy odwołać się do niej za poprzez metodę **GetValue<>**, gdzie używamy typu wartości, np tak:

```csharp
var left = Configuration.GetValue<int>("App:MainWindow:Left", 80);
```

#### Global option class ####

Tu już jest coraz fajniej. Możemy zdefiniować klasę, która będzie odwzorowywać ustawienia aplikacji! A to za sprawą serializacji. Np.:

```csharp
public class MyOptions
{
    public MyOptions()
    {
        // Set default value.
        Option1 = "value1_from_ctor";
    }
    public string Option1 { get; set; }
    public int Option2 { get; set; } = 5;
}
```

A potem już tylko przy konfiguracji serwisów dodajemy wpis:

```csharp
services.Configure<MyOptions>(Configuration);
```

Od tej pory następuje deserializacja ustawień na określony typ. A za pomocą DI możemy w Controllerach i innych klasach wstrzyknąć zależność za pomocą takiego kodu:

```csharp
private readonly MyOptions _options;

public HomeController(IOptions<MyOptions> optionsAccessor)
{
    _options = optionsAccessor.Value;
}
```

#### Section options class ####

Bardzo podobne do powyższego ale możemy tutaj zmapować jedynie część konfiguracji. A wystarczy zrobić to za pomocą odwołania do sekcji:

```csharp
services.Configure<TimeOptions>(Configuration.GetSection("Time"));
```

Od tego momentu ponownie możemy wstrzyknąć opcje z tym modelem i to znów zadziała.

Obie powyższe metody mają cechę wspólną. Są odporne na błędy podczas kodowania. Oczywiscie ustawienia muszą sięzgadzać ale to wciąż duży krok na przód jeśli możemy przechowywać różnego rodzaju typy w ustawieniach i je bezpośrednio odczytywać. Taka konfiguracja typowana.

### Testowanie ###

Konfiguracja może także być w pamięci i to daje nam pole do popisu podczas testów. Nie musimy tworzyć plików konfiguracyjnych a jedynie wystarczy, że przygotujemy słownik z danymi i go zainicjalizujemy. Oto przykład:

```csharp
var dict = new Dictionary<string, string>
{
    {"Profile:MachineName", "Rick"},
    {"App:MainWindow:Height", "11"},
    {"App:MainWindow:Width", "11"},
    {"App:MainWindow:Top", "11"},
    {"App:MainWindow:Left", "11"}
};

var builder = new ConfigurationBuilder();
builder.AddInMemoryCollection(dict);
Configuration = builder.Build();

Console.WriteLine($"Hello {Configuration["Profile:MachineName"]}");

var window = new MyWindow();
Configuration.GetSection("App:MainWindow").Bind(window);
Console.WriteLine($"Left {window.Left}");
```

## Podsumowanie ##

Moim zdaniem - postarali się. Generalnie dali dużo możliwości i to w dość prostej postaci. Ja jestem bardzo zadowolony i daje mi coraz większą przyjemność takie programowanie. Już następnym razem pokażę jak zaimplementować stan użytkownika w aplikacji desktopowej na electronie wraz z logowaniem i rejestracją.

  [1]: https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration
  [2]: https://docs.microsoft.com/en-us/aspnet/core/security/key-vault-configuration
  [3]: https://docs.microsoft.com/en-us/aspnet/core/api/microsoft.extensions.configuration.configurationbuilder