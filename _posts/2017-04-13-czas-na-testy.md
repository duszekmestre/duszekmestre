---
layout:     post
title:      Czas na testy!
date:       2017-04-13 21:59
summary:    Jak to się mówi - lepiej późno niż wcale. Dlatego przyszedł czas na testy i wprowadzenie w jak największym stopniu pokrycia kodu testami. Jak to zrobić?
tags:       DSP2017 tests TDD xUnit Moq
categories: dsp2017
---

Jak to się mówi - lepiej późno niż wcale. Dlatego przyszedł czas na testy i wprowadzenie w jak największym stopniu pokrycia kodu testami. Jak to zrobić?

## TDD - Test Driven Development ##

Programowanie jest bardzo przyjemnym zadaniem. Dopóki nie zaczną pojawiać się pewne komplikacje zwane bugami (z ang. "robaki"). Takie robaki bardzo lubią pojawiać się w naszym kodzie. Generalnie czym bardziej doświadczony programista tym ich jest mniej. ALE. To nie do końca jest jedynie kwestią coraz lepszego pisania kodu ale też związane jest z przygotowaniem się na najgorsze.

Takie przygotowanie daje nam TDD. Jest to pewna metodyka (zaliczana do zwinnych - agile) tworzenia oprogramowania. Polega ona generalnie na 3 krokach:

1. Piszemy test do metody i go uruchamiamy - NIE DZIAŁA
2. Piszemy kod implementujący metodę i ponownie uruchamiamy test - DZIAŁA
3. Korygujemy test o nową logikę i zaczynamy proces od nowa

Nie ukrywam, że jak dla mnie to podejście jest mega ekstramalne. ALE. Rzeczywiście działa. Ostatnio sam się o tym przekonałem, gdy pisałem metodę do tworzenia użytkownika. Pisząc implementację nie przewidziałem jednego z przypadków, który wyszedł mi dopiero podczas testu. 

No dobrze. Był kawałek teorii - teraz czas na praktykę.

## xUnit, Moq i inne ##

Generalnie testy są pisane we frameworkach. Chociaż oczywiście nie muszą. Bo teoretycznie napisanie aplikacji w konsoli, która wywołuje określone metody - też jest pewnym rodzajem testu. Jednakże jeśli mówimy o testach jednostkowych to zdecydowanie o wiele praktyczniejsze jest pisanie ich w określonym frameworku. Na rynku mamy ich wiele:

1. **MSTest** - framework od Microsoftu
2. **NUnit** - jeden z najpopularniejszych frameworków
3. **xUnit** - kolejny mega popularny framework
4. ... i inne

Ja zdecydowałem się na [xUnit][1]. Dlaczego? Sam nie wiem. Ostatnio troszkę poczytałem i generalnie ludzie o wiele bardziej chwalą sobie NUnit i możliwe, że przeprawię swoje testy na NUnit. Ale najpierw będę potrzebował szerszego spectrum w tym temacie.

### Pierwszy test ###

```csharp
[Fact]
public async Task Validate_ReturnsBadRequest_ForModelInvalid()
{   
    // Arrange
    var moqUnitOfWork = new Mock<IUnitOfWork>();
    var moqCache = new Mock<IDistributedCache>();
    var moqSmsSender = new Mock<ISmsSender>();
    var moqCodeGen = new Mock<ICodeGenerator>();

    var accountController = new AccountController(moqUnitOfWork.Object, moqCache.Object, moqSmsSender.Object, moqCodeGen.Object);

    accountController.ModelState.AddModelError("ERROR", "ERROR VALUE");

    // Act
    var result = await accountController.ValidateAsync(new ValidateAccountVM());

    Assert.IsType<BadRequestObjectResult>(result);
}
```

Po komentarzach widać już pierwszą rzecz, którą warto kierować się podczas pisania testu AAA. 

1. **A** - Arrange - tworzymy wszystkie potrzebne obiekty, moki, itp.
2. **A** - Act - wywołanie metody
3. **A** - Assert - sprawdzenie rezultatów

Mój pierwszy test sprawdzać będzie, czy jeśli podamy nieprawidłowy model to aplikacja zwróci rezultat typu *BadRequestObjectResult*. 

### Moq ###

[Moq][2] jest bardzo ciekawą biblioteczką do tworzenia obiektów określonego typu i symulujących ich działanie. Powyższy przykład pokazał jedynie bardzo proste wkorzystanie. Jednakże w innych testach potrzebowałem sprawdzić też troszkę więcej:

```csharp
[Fact]
public async Task Confirm_ReturnsBadRequest_ForInvalidAccountId()
{
    // Arrange
    var testReq = new ConfirmAccountVM
    {
        AccountId = 1,
        ConfirmationCode = ""
    };

    var moqAccountRepo = new Mock<IAccountRepository>();
    moqAccountRepo
        .Setup(repo => repo.GetAsync(testReq.AccountId))
        .Returns(Task.FromResult<Account>(null));
    var moqUnitOfWork = new Mock<IUnitOfWork>();
    moqUnitOfWork
        .SetupGet(x => x.AccountRepository)
        .Returns(moqAccountRepo.Object);
    var moqCache = new Mock<IDistributedCache>();
    var moqSmsSender = new Mock<ISmsSender>();
    var moqCodeGen = new Mock<ICodeGenerator>();

    var accountController = new AccountController(moqUnitOfWork.Object, moqCache.Object, moqSmsSender.Object, moqCodeGen.Object);

    // Act
    var result = await accountController.ConfirmAsync(testReq);

    // Assert
    Assert.IsType<BadRequestObjectResult>(result);

    moqAccountRepo.VerifyAll();
    moqUnitOfWork.VerifyAll();
}
```

Tutaj sekcja *Arrange* jest już o wiele bardziej złożona. Po pierwsze utworzyłem moq dla repozytorium i chciałem aby dla określonego AccountId został zwrócony null. W tym wypadku "zsetupowałem" (piękny polish język :P) wywołanie metody z parametrem i określiłem, że ma zwrócić null.

Na końcu - w sekcji *Assert* wywołałem metodę **VerifyAll()** na moim moq. Co ona robi? Sprawdza, czy napisane przeze mnie konfiguracje zostały wywołane. Dzięki temu dowiem się, czy kod wywołał się tak, jak tego oczekiwałem.

Jeszcze jeden przykład:

```csharp
var testAccount = new Account();

var moqAccountRepo = new Mock<IAccountRepository>();
moqAccountRepo
    .Setup(repo => repo.GetAsync(It.IsAny<int?>()))
    .Returns(Task.FromResult(testAccount));
moqAccountRepo
    .Setup(repo => repo.Update(testAccount));
```

Tutaj można zauważyć, że przekazałem do metody dziwny parametr: **Is.IsAny<int?>()**. Oznacza to, że ten parametr może być dowolną liczbą lub null'em. Ta metoda zwróci nam wtedy określony obiekt (wcześniej utworzony). Natomiast druga metoda już musi ten obiekt przyjąć. Teraz jeśli np okaże się, że referencje do obiektu by się nie zgadzały to podczas asercji dostalibyśmy błąd.

#### Moq - wymagania ####

Generalnie wymagania Moq na początku sprawiły dla mnie wiele problemów. Ale to właśnie dlatego, że najpierw miałem napisany kod - a dopiero potem pisałem testy. Dopiero podczas testów dowiedziałem się np o tym, że moq najlepiej działa na interfejsach. A jeśli już ma wykorzystać klasę to metody, które chcemy użyć muszą być wirtualne. Moq w pamięci tak na prawdę tworzy jakby swoje assembly i tworzy klasy proxy z metodami (buduje metody w pamięci za pomocą Emit i IL). Dlatego jeśli chcemy wykorzystać jakąś metodą - musi ona być wirtualna, żeby Moq mógł ją nadpisać. Dodatkowo nie da się nadpisać metod statycznych, co też spowodowało u mnie komplikacje z przetestowaniem cache, w którym używam extensions metod (one są statyczne). Ale na szczęście obyło się ostatecznie bez tego.

## Podsumowanie ##

Etap pierwszego ekraniku był dość długi. Wydaje się, że przecież to tylko logowanie i rejestracja. Ale pociągnął on za mną wiele decyzji i potrzebę przygotowania powoli core projektu pod kolejne wymagania. A to już jest czasochłonne. Wiele rzeczy z tego wszystkiego też poznaję, bo ASP.NET Core, Angular 4, webpack, electron są dla mnie pewną nowością, z którą wcześniej miałem do czynienia jedynie podczas prezentacji lub krótkich testów. Teraz mierzę się już z konkretnymi wymaganiami i funkcjonalnościami. Po przebrnięciu przez początki mogę powiedzieć, że coraz bardziej mi się to podoba! 

Zazwyczaj piszę co dalej. To i dziś napiszę :) Muszę zaplanować jak będzie wyglądać główne okno aplikacji i jakie funkcjonalności będą już w komunikatorze. Ale przed tym zajmęsię najpierw przekazaniem tokenu użytkownika do okna głównego i napiszę serwis w angularze do przechowywania stanu użytkownika podczas uruchomienia wraz z zapisem i odtworzeniem.

Jako iż Wielki Czwartek - to wszystkim obchodzącym święta Wielkiej Nocy życzę spokojnych, zdrowych i wesołych świąt. Odpoczynku i refleksji.


  [1]: https://xunit.github.io/
  [2]: https://github.com/moq/moq4