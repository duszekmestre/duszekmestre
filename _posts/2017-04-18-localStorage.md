---
layout:     post
title:      localStorage - magazyn w przeglądarce
date:       2017-04-18 19:42
summary:    Często potrzebujemy po stronie klienta przechowywać różnego rodzaju dane, które aplikacja internetowa może potem wykorzystywać. Ciasteczka niezbyt do tego się nadają. Z pomocą przychodzi nam localStorage.
tags:       DSP2017 localStorage HTML5

---

Często potrzebujemy po stronie klienta przechowywać różnego rodzaju dane, które aplikacja internetowa może potem wykorzystywać. Ciasteczka niezbyt do tego się nadają. Z pomocą przychodzi nam localStorage.

## Czym jest localStorage? ##

Na stronie w3schools możemy przeczytać kilka zdań o [localStorage][1]. Parafrazując:

> Za pomocą lokalnego magazynu, aplikacje internetowe mogą przechowywać dane w przeglądarce użytkownika. 
> Przed HTML5, dane aplikacji musiały być przygotowywane w ciasteczkach i powinny być zawarte w każdym żądaniu do serwera. Magazyn lokalny jest bezpieczniejszy i może przechowywać znacznie więcej danych bez wpływu na wydajność.

Najważniejszym punktem tego było dla mnie to, że jest bezpieczniejszy i nie wymaga się aby każdorazowo dbać o niego. Wystarczy raz zapisać i już mamy to dostępne dla naszej "strony", czyli aplikacji. Plusem (choć w moim wypadku obojętnym) jest wsparcie dla localStorage w każdej współczesnej przeglądarce. Chociaż akurat electron i tak jest zbudowany na chromie. LocalStorage jest też o tyle dobre, że przechwywane jest także po zamknięciu przeglądarki.

### Jak skorzystać? ###

localStorage jest jak słownik. Aby dodać element do tego słownika używamy metodę **setItem**:

```js
localStorage.setItem("key", "value");
```

Natomiast jeśli chcemy uzyskać wartość z klucza wywołujemy metodę **getItem**:

```js
var value = localStorage.getItem("key");
```

Gdybyśmy chcieli usunąć wartość z klucza, możemy użyć metody **removeItem**:

```js
localStorage.removeItem("key");
```

Warto pamiętać, że klucz i wartość są przechowywane jako stringi więc jeśli chcemy przechowywać tam obiekt - musimy go najpierw zserializować a przy odczycie zdeserializować.

### A gdzie expiration date? ###

No właśnie. Tego brakuje. Przynajmniej z mojej perspektywy. ALE. Jak wrzystko - da się obejść. W związku z czym napisałem własną klasę, która na bazie localStorage może wrzucać klucze z pseudo wygaśnięciem:

```js
export class PersistenceCache {
    static cache: Storage = localStorage;

    static setItem(key: string, value: string, expirationDate?: Date) {
        var cacheObj = new CacheObject();
        cacheObj.value = value;
        cacheObj.expireDate = expirationDate;

        PersistenceCache.cache.setItem(key, JSON.stringify(cacheObj));
    }

    static getItem(key: string) {
        var cacheJson = PersistenceCache.cache.getItem(key);
        if(cacheJson) {
            var cacheObject = JSON.parse(cacheJson);
            var expDate = cacheObject.expireDate;
            var now = new Date();

            if (expDate && expDate < now){
                PersistenceCache.cache.removeItem(key);
                return null;
            }

            return cacheObject.value;
        }
    }

    static removeItem(key: string) {
        PersistenceCache.cache.removeItem(key);
    }
}

class CacheObject {
    value: string;
    expireDate?: Date;
}
```

Generalnie zasada opiera się na wygaśnięciu sprawdzanym podczas odczytu. Czyli jeśli okaże się, że data wygaśnięcia już minęła to w tym momencie usuwamy dany obiekt i zwracamy **null**;

## Co dalej? ##

No tak. Co dalej? Odwieczne pytanie :) Bez planu nie da się nic dalej zrobić, dlatego kolejnym krokiem będzie planowanie. I o tym planowaniu troszkę w kolejnym poście. Najpierw opiszę kilka kroków jak sam się do tego zabieram, następnie zaplanuję sobie rzeczy do zrobienia i ich kolejność oraz wybiorę inspirację do UI aplikacji.

Do następnego razu!


  [1]: https://www.w3schools.com/html/html5_webstorage.asp