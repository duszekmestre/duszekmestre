---
layout:     post
title:      Repository Pattern & Unit Of Work w .NET Core
date:       2017-03-26 19:56
summary:    Wzorzec repozytorium oraz jednostka pracy - dziwne polskie tłumaczenie, które jednak ma w sobie wiele znaczenia. Dzisiejszy wpis bedzie o implementacji generycznego rozwiazania dla wzorców dostępu do danych.
tags:       DSP2017 repository unitofwork efcore efmigrations
categories: dsp2017
---

Wzorzec repozytorium oraz jednostka pracy - dziwne polskie tłumaczenie, które jednak ma w sobie wiele znaczenia. Dzisiejszy wpis bedzie o implementacji generycznego rozwiazania dla wzorców dostępu do danych.

## Repository pattern ##

Repozytorium - czyli miejsce składowania danych. A w informatyce to oznacza bardziej zbiór danych i dostęp do nich. Samo przechowywanie w tym momencie nas tak bardzo nie dotyka i zależeć powinno od konfiuracji (taka warstwa abstrakcyjna). Z racji, ze poszedłem w ASP.NET Core Web API to pomyślałem, że nie będę też wydziwiał (a może właśnie będę :P) i jako bibliotekę dostępu do danych użyje [Entity Framework Core][1]. Ta biblioteka jest typu ORM (Object-relational mapper) i odcina dewelopera od pisania połączenia z danymi. Jedynie operuje na danych tak, jakby były w pamięci. Dodatkową zaletą EF Core jest w tym momencie jego rozszerzalność. W tym momencie zapewnia dostęp do baz danych, takich jak:

1. MS SQL
2. SQLite
3. MySQL
4. PostgreSQL
5. InMemory
6. Devart
7. inne

W tym inne zawiera się najbardziej to, że można napisać własnego providera.

### Instalujemy ###

Tak w sumie to wielkiej instalacji tutaj nie będzie :) W naszej solucji dodałem projekt MassCo.Data, który jest typu Class Library. Do obu projektów następnie dodałem trzy biblioteki:

```
Install-Package Microsoft.EntityFrameworkCore.InMemory
```

```
Install-Package Microsoft.EntityFrameworkCore.SqlServer
```

```
Install-Package Microsoft.EntityFrameworkCore.Tools
```

### Generyczne repozytorium ###

Zamiast pisać kilka/dziesiąt repozytoriów per klasę warto napisaćjedno generyczne. Oczywiście - nie jest w stanie ono zapewnić wszystkcih operacji w naszej aplikacji, ale daje dużą podstawę. Oto kod takiej generycznej klasy:

```csharp
public class GenericRepository<T> : IGenericRepository<T>
    where T : class, IEntity, new()
{
    protected DbContext dbContext;
    private DbSet<T> dbSet;

    public GenericRepository(DbContext context)
    {
        this.dbContext = context;
        this.dbSet = dbContext.Set<T>();
    }

    public async Task<T> GetAsync(int? id)
    {
        var entity = await dbSet.FirstOrDefaultAsync(x => x.Id == id);
        return entity;
    }

    public async Task<IEnumerable<T>> GetAllAsync()
    {
        var entities = await dbSet.ToListAsync();

        return entities;
    }

    public void Create(T entity)
    {
        dbSet.Add(entity);
    }

    public void Update(T entity)
    {
        dbContext.Entry(entity).State = EntityState.Modified;
    }

    public void Delete(T entity)
    {
        dbSet.Remove(entity);
    }

    public async Task DeleteAsync(int id)
    {
        var entity = await this.GetAsync(id);
        if (entity != null)
        {
            dbSet.Remove(entity);
        }
    }
}
```

Lubię pisać kodem :) Ale teraz kilka słów wyjaśnienia. Sama klasa wykorzystuje zbiór danych określonego typu. Widać to już w konstruktorze. Implementuje ona interfejs:

```csharp
public interface IGenericRepository<T>
    where T : class, IEntity, new()
{
    Task<T> GetAsync(int? id);
    void Create(T entity);
    Task DeleteAsync(int id);
    void Delete(T entity);
    void Update(T entity);
    Task<IEnumerable<T>> GetAllAsync();
}
```

Czyli główne operacje na danych, takie jak pobranie pojedynczego elementu, stworzenie, aktualizacja, usunięcie, czy pobranie wszystkcih elementów w kolekcji. Generalnie ta ostatnia metoda na ten moment nie ma jeszcze parametrów ALE musi mieć. I nie dlatego, że nie zadziała. Ale po prostu będzie ona działać jedynie do czasu. Dlatego przy najbliższej okazji zmieni ona swoją implementację.

### Kontekst danych ###

Wszystkie dane pracuję w pewnym kontekscie. Ten kontekst, to połączenie. Zawsze trzeba napisać swój. Nawet pusty. To wymóg. U nas dodatkowo w kontekście będziemy inicjalizować kontekst zbiorami danych dla typów, które oznaczymy interfejsem IEntity. Takze nasz kontekst wygląda tak:

```csharp
public class GenericDbContext : DbContext
{
    private static bool created = false;
    public GenericDbContext(DbContextOptions options)
        : base(options)
    {
        if (!created)
        {
            Database.EnsureCreated();
            created = true;
        }
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        var iEntityType = typeof(IEntity);

        var modelAssembly = iEntityType.GetTypeInfo().Assembly;

        var types = modelAssembly.GetTypes()
            .Where(x => iEntityType.IsAssignableFrom(x) && x.GetTypeInfo().IsClass);

        var method = typeof(ModelBuilder).GetMethods().First(m => m.Name == "Entity"
            && m.IsGenericMethodDefinition
            && m.GetParameters().Length == 0);

        foreach (var type in types)
        {
            method = method.MakeGenericMethod(type);
            method.Invoke(modelBuilder, null);
        }

        base.OnModelCreating(modelBuilder);
    }
}
```

W konstruktorze mamy sprawdzenie, czy baza danych została utworzona. Jeśli nie to automatycznie wyrzuci wyjątkiem. Druga ważniejsza kwestia to właśnie rejestracja zbiorów danych za pomocą nadpisanej metody OnModelCreating. Domyślnie ta metoda działa tak, że wykorzystuje właściwości w kontekście, które są typu **DbSet<T>**. My jednak piszemy generycznie i nie chcemy za każdym razem dodawać właściwości do klasy. Sam kod polecam przeanalizować samemu. Troszkę refleksji i poszło! :)

### Unit of Work ###

Jednostka pracy. Po co jest? Ona grupuje operacje. Dzięki niej będziemy posługiwać się zawsze nowym połączeniem a wszelkie operacje będą w nim zgrupowane i wywoływane. Dodatkowo późniejsza implementacja zapewni nam też transakcyjność operacji. A jak to wygląda? Proszę:

```csharp
public class UnitOfWork : IUnitOfWork
{
    private GenericDbContext Context;
    private GenericRepository<Account> accountRepository;

    public IGenericRepository<Account> AccountRepository
    {
        get
        {
            return accountRepository = accountRepository ?? new GenericRepository<Account>(Context);
        }
    }

    public UnitOfWork(GenericDbContext dbContext)
    {
        this.Context = dbContext;
    }

    public void Save()
    {
        this.Context.SaveChanges();
    }

    public async Task SaveAsync()
    {
        await this.Context.SaveChangesAsync();
    }
}
```

Generalnie taka jednostka zawiera dostęp do różnych repozytoriów. Zabezpiecza nas za to przed bezpośrednim korzystaniem z kontekstu. Dzięki temu wykorzystamy jedynie te repozytoria, które nam udostępni jednostka pracy. 

### Rozruch - DI ###

Ekhm. No tak. Ale skoro piszemy w Core to piszmy dobrze i wykorzystajmy kolejną wbudowaną rzecz. DI - dependency injection - wstrzykiwanie zależności. Aby wstrzyknąć jakąś zależność do innej klasy to musmy najpierw taką zależność zarejestrować. Idziemy więc do klasy **Startup** i dopisujemy taki oto kod:

```csharp
private void ConfigureDatabase(IServiceCollection services)
    {
        var useInMemoryDatabase = Configuration.GetSection("Settings").GetValue<bool>("UseInMemoryDatabase");

        services.AddDbContext<GenericDbContext>(options =>
        {
            if (useInMemoryDatabase)
            {
                options.UseInMemoryDatabase();
            }                           
            else
            {
                options.UseSqlServer(Configuration.GetConnectionString("MassCoLocalDB"));
            }
        });
        services.AddSingleton<IUnitOfWork, UnitOfWork>();
    }
```

Tutaj widzimy, że można wykorzystać ustawienia do inicjalizacji kontekstu. Wskazujemy, że chcemy aby nasz kontekst danych był typu **GenericDbContext** i w zależnosci od ustawień w pliku **appsettings.json** wykorzystywać dane z pamięci lub z SQL servera. Dodatkowo na samym końcu tej metody widzimy, że pojawiła się linijka

```csharp
services.AddSingleton<IUnitOfWork, UnitOfWork>();
```

Dzięki temu wskazujemy, że chcemy aby klasa jednostki pracy była singletonem. Póki co tak jest ok. Z czasem zastanowić się będzie trzeba czy przypadkiem taka klasa nie powinna jednak mieć krótszego życia.

Teraz sostało nam juztylko w metodzie *void ConfigureServices(IServiceCollection services)* dodać wpis na początku:

```csharp
ConfigureDatabase(services);
```

### Działanie ###

A jak to działa? Proste. Na początek stworzyłem kontroler **AccountController**. W nim kilka metod CRUD:

```csharp
[Route("api/[controller]")]
public class AccountController : Controller
{
    public IUnitOfWork UnitOfWork { get; private set; }

    public AccountController(IUnitOfWork unitOfWork)
    {
        this.UnitOfWork = unitOfWork;
    }

    // GET: api/values
    [HttpGet]
    public async Task<IEnumerable<Account>> GetAsync()
    {
        return await UnitOfWork.AccountRepository.GetAllAsync();
    }

    // GET api/values/5
    [HttpGet("{id}")]
    public async Task<Account> GetAsync(int id)
    {
        return await UnitOfWork.AccountRepository.GetAsync(id);
    }

    // POST api/values
    [HttpPost]
    public async Task PostAsync([FromBody]Account account)
    {
        UnitOfWork.AccountRepository.Create(account);
        await UnitOfWork.SaveAsync();
    }

    // PUT api/values/5
    [HttpPut("{id}")]
    public async Task PutAsync(int id, [FromBody]Account account)
    {
        var entity = await UnitOfWork.AccountRepository.GetAsync(id);
        if (entity == null)
        {
            return;
        }          

        entity.Name = account.Name;
        entity.PhoneNumber = account.PhoneNumber;

        UnitOfWork.AccountRepository.Update(entity);
        await UnitOfWork.SaveAsync();
    }

    // DELETE api/values/5
    [HttpDelete("{id}")]
    public async Task DeleteAsync(int id)
    {
        await UnitOfWork.AccountRepository.DeleteAsync(id);
        await UnitOfWork.SaveAsync();
    }
}
```

W szczegóły każdej metody nie będę wnikał. Jedynie powiem, że samo wstrzykiwanie zależności odbywa się poprzez konstruktor. Tam włąśnie IoC wrzuca nam zarejestrowaną zależność, z której potem mozemy korzystać. Powiem szczerze, że jak napisałem ten kod, to byłem pod wrażeniem, że to wszystko po prostu zadziałało!

Przetestowanie kontrolera pozostawiam każdemu z osobna. Podpowiadam - PostMan.

## Migracje ##

Dla tych, którzy myśleli, że to koniec - niestety nie. Aby dokonać uruchomienia potrzeba miejsca na dane i zainicjalizowanie bazy naszą tabelką. Generalnie do zmian w strukturze bazy dancyh wykorzystać można migracje. Ja wykorzystam wbudowane w EF Core. To właśnie do nich potrzebna była paczka *Microsoft.EntityFrameworkCore.Tools*. Skoro mamy stworzoną klasę Account (ona będzie miejscem dla kont naszych użytkowników), możemy przystąpić do konfiguracji. Ja dla testów i środowiska deweloperskiego nie instalowałem w sumie nic. Dodałem jedynie w konfiguracji wpis:

```json
{
    "ConnectionStrings": {
        "MassCoLocalDB": "Server=(localdb)\\mssqllocaldb;Database=MassCo.Database;Trusted_Connection=True;"
    },
    "Settings": {
        "UseInMemoryDatabase": false 
    }
}
```

Koniecznie przed uruchomieniem migracji musicie ustawić false przy *UseInMemoryDatabase*. Dlaczego? Podczas uruchomienia migracji, silnik sprawdza na początek gdzie ma ją wykonać. Wykorzystuje do tego metodę Configure naszej aplikacji. A tam jest logika sterująca na podstawie tego ustawienia. Gdyby to ustawienie było na true to byśmy mieli błędy. Także skoro jest wszystko skonfigurowane to już tylko jeden krok (ja go wykonałem dla projektu MassCo.Data):

```
Add-Migration InitialMigration
```

Dzięki temu utworzyła nam się pierwsze migracja, która zainicjalizowała wszystkie zbiory danych (tabele) w naszej bazie. Teraz aby uruchomićmigracje wystarczy tylko:

```
Update-Database
```

## Podsumowanie ##

Skoro mamy juz repozytorium i jednostkę pracy a także migracje, miejsce na dane i dostęp do nich - wystarczy tylko działać. Tyle, że jeszcze nas troszkę czeka. W kolejnym tygodniu planuję token based authentication podpiąć pod bazę oraz dodać rejestrację użytkowników z użyciem kodów SMS. Zachęcam do przeglądania zmian w [repozytorium GIT][2] a dziś jeszcze napiszę krótki wpis na blogu z przemyśleniami o pracy zespołowej w rzeczywistej pracy.


  [1]: https://docs.microsoft.com/en-us/ef/
  [2]: https://github.com/duszekmestre/MassCo