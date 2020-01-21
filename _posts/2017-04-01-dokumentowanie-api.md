---
layout:     post
title:      Dokumentowanie API
date:       2017-04-01 20:32
summary:    API - Application Programming Interface. Interfejs - czyli coś co wymaga instrukcji. I najlepszą instrukcją jest dobra dokumentacja. A jak zrobić dobrą dokumentację przy okazji wiele nad tym się nie napracując? Zobaczcie sami.
tags:       DSP2017 swagger documentation asp.net core

---

API - Application Programming Interface. Interfejs - czyli coś co wymaga instrukcji. I najlepszą instrukcją jest dobra dokumentacja. A jak zrobić dobrą dokumentację przy okazji wiele nad tym się nie napracując? Zobaczcie sami.

### Swagger ###

Od razu przejdźmy do sedna. [Swagger][1] - w sumie to jest wielka rzecz. To taki framework, który umożliwia projektowanie, tworzenie i dokumentowanie API. W ASP.NET Core istnieje natomiast bardzo prosta metoda do tego aby istniejące API udokumentować. Wystarczy paczka **Swashbuckle.AspNetCore**. Dlatego dodajmy ja do naszego projektu:

```
Install-Package Swashbuckle.AspNetCore
```

A potem już tylko kilka linijek w klasie Startup. Najpierw metoda *ConfigureServices*:

```csharp

services.AddSwaggerGen(c =>
{
    c.SwaggerDoc("v1", new Info
    {
        Title = "MassCo API",
        Version = "v1",
        Contact = new Contact
        {
            Name = "duszekmestre",
            Url = "https://duszekmestre.github.io/"
        }
    });

    var basePath = PlatformServices.Default.Application.ApplicationBasePath;
    var xmlPath = Path.Combine(basePath, "MassCo.API.xml");
    c.IncludeXmlComments(xmlPath);
});

```

Do serwisów dodaliśmy generator Swaggera wraz z konfiguracją. To jest nasza pierwsza wersja dlatego v1 + informacje kontaktowe, które będą wyświetlanie na stronie dokumentacji. Kolejna konfiguracja służy do tego, aby Swagger podczas generowania wziął pod uwagę komentarze dokumentujące z kodu. We właściwościach projektu należy zaznaczyć tak jak poniżej:

![Project properties][3]

Dzięk itemu po uruchomieniu projektu wygeneruje się z naszych komentarzy plik XML, który zostanie użyty przez Swaggera. Ale do uruchomienia samego Swaggera potrzebna jest jeszcze jedna rzecz w metodzie *Configure*:

```csharp
app.UseSwagger();

app.UseSwaggerUI(c =>
{
    c.SwaggerEndpoint("/swagger/v1/swagger.json", "MassCo API V1");
    c.RoutePrefix = "docs";
});
```

I to w sumie wszystko. Teraz wystarczy, że uruchomimy projekt i przejdziemy pod ścieżkę **/docs/** i mamy dokumentację:

![Swagger generated docs][4]

### Podsumowanie ###

Jak sami widzicie - ciężko nie jest. Wiele kodu nie było trzeba napisać. Oczywiscie aby dokumentacja była jak najlepsza warto dodawać komentarze dokumentujące do kodu takie jak ten:

```csharp
/// <summary>
/// POST api/account
/// Register/create new account
/// </summary>
/// <param name="request">Required phone number and username</param>
/// <returns>Newly created item</returns>
/// <response code="201">Successfuly created item</response>
/// <response code="400">If request has errors</response>
/// <response code="409">If phone number already exists</response>
[HttpPost]
[ProducesResponseType(typeof(Account), 201)]
[ProducesResponseType(400)]
[ProducesResponseType(typeof(string), 409)]
public async Task<IActionResult> PostAsync([FromBody]RegisterAccountVM request)
```

Po więcej informacji i szczegółowy opis instalacji i dokumentowania zapraszam pod adres: [https://docs.microsoft.com/en-us/aspnet/core/tutorials/web-api-help-pages-using-swagger][2]. Dowiecie się na niej dodatkowo jak zmienić style strony i kilka ciekawostek z dokumentowanie modelu.

Kolejny post dotyczyć będzie ponownie logowania i rejestracji. Do usłyszenia!


  [1]: http://swagger.io/
  [2]: https://docs.microsoft.com/en-us/aspnet/core/tutorials/web-api-help-pages-using-swagger
  [3]: {{ site.baseurl }}/images/properties-xml-option.PNG
  [4]: {{ site.baseurl }}/images/swagger-docs.PNG