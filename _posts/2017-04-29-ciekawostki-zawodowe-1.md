---
layout:     post
title:      Ciekawostki zawodowe [v1]
date:       2017-04-29 21:41
summary:    Dzisiaj troszkę odmienny klimat wpisu. Opowiem o ciekowstce, którą ostatnio spotkałem w swojej pracy. Mam nadzieję, że Wam również rozwiązanie kiedyś się przyda.
tags:       DSP2017 stacktrace .net

---

Dzisiaj troszkę odmienny klimat wpisu. Opowiem o ciekawostce, którą ostatnio spotkałem w swojej pracy. Mam nadzieję, że Wam również rozwiązanie kiedyś się przyda.

## StackTrace - gdzie jestem? ##

Od razu przejdę do rzeczy. Mieliśmy przypadek związany z logowaniem. Po krótcę tak go zademonstruję:

```csharp
class Program
{
    static void Main(string[] args)
    {
        Console.WriteLine(A());

        Console.ReadLine();
    }

    public static string A()
    {
        return B();
    }

    private static string B()
    {
        return Environment.StackTrace;
    }
}
```

Wynik? (Ups - po polsku :P)

```
w System.Environment.GetStackTrace(Exception e, Boolean needFileInfo)
w System.Environment.get_StackTrace()
w StackTrace.Program.B() w c:\users\dusze\documents\visual studio 2017\Projects\Work\StackTrace\Program.cs:wiersz 21
w StackTrace.Program.A() w c:\users\dusze\documents\visual studio 2017\Projects\Work\StackTrace\Program.cs:wiersz 16
w StackTrace.Program.Main(String[] args) w c:\users\dusze\documents\visual studio 2017\Projects\Work\StackTrace\Program.cs:wiersz 9
```

Wszystko wydaje się w porządku. Oczywiscie można pominąć te dwa pierwsze wpisy modyfikująć metodę B:

```csharp
private static string B()
{
    return new System.Diagnostics.StackTrace().ToString();
}
```

czego wynikiem będzie:

```
w StackTrace.Program.B()
w StackTrace.Program.A()
w StackTrace.Program.Main(String[] args)
```

Dla ułatwienia - pozostaniemy przy tej drugiej opcji. Wszystko jest ok do czasu. Wrzucamy projekt na produkcję a tam wynikiem powyższego kodu będzie:

```
w StackTrace.Program.Main(String[] args)
```

EEE? Ale jak to? Przecież to niemożliwe! A no właśnie możliwe. Troszkę nam to zajęło ale okazało się, ze kompilator języka .NET w trubie Release podczas kompilacji usuwa tzw. Inlining. Czyli bezpośrednie wywołania metod ograniczając przy tym zbędne wywołania. Dzięki temu mamy o 2 przeskoki mniej! W prostych projektach nie ma to dla nas żadnego znaczenia. Ale z założenia kompilator w trybie Release optymalizuje. Są dwa obejścia:

1. **Wyłączenie optymalizacji** - żart ale prawdziwy :) Nie polecam generalnie bo stracimy wszystkie optymalizacje a nie tylko tą jedną.
2. Dodanie atrybutu do metod, które jednak chcemy aby były w StackTrace

No i właśnie my zastosowaliśmy tą drugą metodę. Dzięki niej mamy kontrolę nad tym, które metody będą wskazane w StackTrace i nie zostaną potraktowane optymalizacją "inliningu". Kolejna modyfikacja metody B wygląda tak:

```csharp
[MethodImpl(MethodImplOptions.NoInlining)]
private static string B()
{
    return new System.Diagnostics.StackTrace().ToString();
}
```

No i oczywiście wynik:

```
w StackTrace.Program.B()
w StackTrace.Program.Main(String[] args)
```

Pięknie!

## Weekend ##

Dlaczego dzisiaj tak mało? No cóż. Majówka. Dlatego lekki temat i dlatego w kolejnym tygodniu nie powstanie żaden nowy wpis. Pozdrawiam i wszystkim miłego odpoczynku!