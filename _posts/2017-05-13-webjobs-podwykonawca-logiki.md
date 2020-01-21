---
layout:     post
title:      WebJobs - podwykonawca logiki
date:       2017-05-13 18:00
summary:    Musimy nieraz wykonać inną ważniejszą pracę i komu zlecić wykonanie tej mniej ważnej abyśmy mieli więcej czasu. Podobnie jest w aplikacjach. Czasami niepotrzebujemy zatrzymywać głównego procesu dla kilku mniej znaczących akcji.
tags:       DSP2017 webjobs hangfire

---

Musimy nieraz wykonać inną ważniejszą pracę i komu zlecić wykonanie tej mniej ważnej abyśmy mieli więcej czasu. Podobnie jest w aplikacjach. Czasami niepotrzebujemy zatrzymywać głównego procesu dla kilku mniej znaczących akcji.

## WebJobs ##

Niestety tutaj troszeczkę ta nazwa będzie myląca, ponieważ samo WebJobs może kojarzyć się niektórym z usługą Azure, która umożliwia wykonywanie zadać rekurencyjnie, jednorazowo lub stale. Jednakże z mojego punktu widzenia każde rozwiązanie, które daje nam takie możliwości jest czymś takim jak WebJobs. W MassCo zastosowałem inne rozwiązanie. Również bardzo dobrze jednakże nie wymaga ono hostowania na Azure.

### Hangfire ###

Bardzo znanym niektórym osobom jest właśnie [Hangfire][1].  Jak sami mówią o swoim produkcie:

> An easy way to perform background processing in .NET and .NET Core applications. No Windows Service or separate process required.
Backed by persistent storage. Open and free for commercial use.

Czyli wszystko pięknie! Dlatego już pokazuję jak w prosty sposób wykorzystać to w naszym projekcie.

### Sms sender ###

Stwierdziłem, że wysyłaniu smsów brakuje jednej rzeczy. Logowania i monitorowania. Hangfire to posiada. Dlatego w kilku prostych krokach go dodaję do projektu. Na początku dodajemy paczkę z nugeta:

```
Install-Package Hangfire
```

Paczka ta automatycznie zainstaluje nam **Hangfire.Core** oraz **Hangfire.SqlServer**. Dzięki temu możemy mieć "persistent storage" w postaci SQLa. Są również inne możliwości, ale o tym możecie poczytać na stronie twórcy.

Teraz czas na konfigurację. W pliku **Startup.cs** dodaję zatem dwa wpisy:

W metodzie *ConfigureServices(IServiceCollection services)*:

```csharp
services.AddHangfire(x => x.UseSqlServerStorage(Configuration.GetConnectionString("WebJobs")));
```

Natomiast w metodzie *Configure(...)*:

```csharp
app.UseHangfireServer(new BackgroundJobServerOptions
{
    Activator = new CoreContainerActivator(serviceProvider)
});
app.UseHangfireDashboard("/tools/jobs");
```

Powyższa konfiguracja nie jest podstawowa, dlatego już wyjaśniam. W konfigurowaniu Dashboardu zmieniłem domyślny adres dashboardu  (domyślnie: "/hangfire").

### IoC i Hangfire ###

W konfiguracji Servera musiałem natomiast nadpisać aktywator jobów. Activator jest potrzebny wtedy, kiedy chcemy aby po naszemu aktywować zadania w tle. A dokładnie aby aktywować instancję zadania. Jako iż mam wprowadzone zależności do projektu to potrzebowałem zrobić to zgodnie z przykładem z dokumentacji i dodać tam IoC. Klasa aktywacyjna wygląda następująco:

```csharp
class CoreContainerActivator : JobActivator
{
    private IServiceProvider serviceProvider;

    public CoreContainerActivator(IServiceProvider serviceProvider)
    {
        this.serviceProvider = serviceProvider;
    }

    public override object ActivateJob(Type jobType)
    {
        return this.serviceProvider.GetService(jobType);
    }
}
```

### Jak używać? ###

Hangfire umożliwia wykonywanie kilku rodzajów akcji:

* **Fire-and-forget jobs** - zadania wywołaj i zapomnij
* **Delayed jobs** - zadanie odroczone o x czasu
* **Recurring jobs** - powtarzalne zadania

W moim przypadku potrzebuję tej pierwszej opcji czyli dodania zadania do kolejki i zapomnienie o wszystkim. Dlatego to robię:

```csharp
BackgroundJob.Enqueue<ISmsSender>(x => x.SendSmsAsync(request.PhoneNumber, $"Your code is: {confirmationCode}"));
```

I to wszystko! Prawda, że proste? I co najważniejsze. Działa bez zarzutu. Teraz wystarczy zajrzeć do dashboardu i zobaczyć status. Wszystko OK. Widać też tam z jakimi parametrami uruchomiono zadanie (numer telefonu i kod), z jakiej maszyny (server) oraz z jakiej kolejki pobrano zadanie oraz kilka statystyk takich jak Opóźnienie, czas trwania i status.

## Co dalej? ##

Ze względu na taką dłuższą przerwę będę miał co nadrabiać w aplikacji. Teraz niestety zajmie mi sporo czasu pisanie widoków w aplikacji (bez ich oprogramowani :(). Ale na to też niestety trzeba czas poświęcić. Ciężko coś opisywać w związku z tym ale będę starał się zamieszczać także wpisy na tematy troszkę oddalone od projektu jeśli w samym projekcie nic ciekawego się nie pojawi :) 

Do usłyszenia!


  [1]: https://www.hangfire.io/