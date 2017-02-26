---
layout:     post
title:      Blog na GitHubie
date:       2017-02-27 07:00
summary:    Przy ostatnim poście mocno się namęczyłem aby Wam przekazać te wszystkie informacje. Ale jednocześnie dzięki całemu temu czasowi odkryłem fajne sposoby na przyszłość. Oto kilka kroków ku temu aby pisanie bloga stało się o wiele prostsze.
categories: DSP2017 MassCo GitHub Jekyll blog
---

Przy ostatnim poście mocno się namęczyłem aby Wam przekazać te wszystkie informacje. Ale jednocześnie dzięki całemu temu czasowi odkryłem fajne sposoby na przyszłość. Oto kilka kroków ku temu aby pisanie bloga stało się o wiele prostsze.

## Laverna - markdown notepad ##
Zauważyłem ostatnio, że bardzo ciężko mi się pisze na różnych blogach a to głównie dlatego, że zamiast skupić się na pisaniu to wciąż skupiam na narzędziu. Ono mnie rozprasza. Zrobiłem zatem mały research. Wiedziałem już wcześniej, że istnieje coś takiego jak markdown. Markdown jest składnią pisania tekstu, która przy zastosowaniu narzędzi kowertuje prosty tekst do HTML'a. Po więcej szczegółów zapraszam na [tą stronę][1]. Generalnie jeśli ktoś wcześniej miał do czynienia z LaTEX to będzie wiedział o co mniej więcej chodzi z markdown. Generalnie cel jest ten sam.

Do edycji markdown nie trzeba używać zaawansowanych notatników. Wystarczy najzwyklejszy notatnik z Windowsa. ALE. Jak z każdym jezykiem programowania lub czymkolwiek bardziej zaawansowanym niż sam tekst warto korzystać z edytorów. Mimo iż kod piszę zazyczaj w Visual Studio to jest on zdecydowanie za ciężki. Wersja VS Code natomiast już jest lekka i zawiera podpowiedzi składni Markdown. Jednakże wciąż mi tam brakuje czegoś takiego jak Live Preview. Jak wiecie zaczynam swoją przygodę z Electronem. Toteż przeglądając aplikacje wykorzystujace framework Electron napotkałem kilka [edytorów Markdown][2]. Generalnie wszystkie wydają się być spoko. Mnie natomiast przyciągnął edytor o nazwie [Laverna][3]. Generalnie jest to nie tyle edytor co notatnik, w którym mozna używać skłądni Markdown. Posiada dodatkowo kilka funkcji:

 1. Edycja z podglądem na żywo
 2. Tworzenie notesów (katalogów)
 3. Tagowanie notatek
 4. Podstawowe skróty klawiaturowe do formatowania tekstu
 5. Podświetlanie składni
 6. Tworzenie list TODO
 7. Import/export do pliku (niestety brak eksportu do HTML)
 8. Synchronizacja z Dropbox/RemoteStorage
  
Póki co wykorzystuję ok połowy  funkcji. Ale kto wie. Może z czasem będzie tego więcej :)

## A co z blogiem? ##
No tak. Edytor wybrany... Ale co z blogiem? No i tutaj generalnie geneza powinna być zgoła inna. Najpierw wybór dobrego bloga, a potem dostosowanie się z tekstem do niego. Ale jak to ze mną bywa - lubię robić na opak. Ostatni post przy użyciu markdown napisałem w mniej niż godzinę. Problemem napotkałem dopiero, gdy napisany tekst musiałem wrzucić na dotychczasowego bloga. Niestety mają oni swoją własną składnię. Więc aczałem znów prowadzić research. Poszukiwałem bloga, który da mi możliwość wrzucania treści w składni markdown. 
...
Co się okazało? Są tylko wyjątki, które to umożliwiają. Pierwszy z nich to platforma [Ghost][4]. Wszystko byłoby spoko ale 19$ miesięcznie troszkę mi się nie widzi. :/

Kolejny przykład to [logdown][5]. I w sumie było wszystko spoko... Ale troszkę sie bałem, że to średnio popularny serwis, który może w każdej chwili zniknąć i mój blog wraz z nim... Po prostu wewnętrzne przeczycie mówiło mi - daj sobie z tym spokój.

Ostatnim bastionem, który mi się przewijał nonstop podczas researchu był... [github][6]! Okazuje się, że nie tylko mozemy tam hostować swoje repozytoria ale takze mieć własną stronę. Może nie tyle własną co z założenia chodzi o stronę własnego projektu. I w sumie tam jest wszystko to, czego potrzebowałem. Github pod spodem wykorzystuje Jekyll (silnik do konwersji skłądni Markdown do stron statycznych/blogów). Rozwiazanie jak dla mnie - idealne!

## Krok po kroku ##
W sumie już na samej stronie GitHub Pages mamy opisane wszystko. Ale ja akurat poszedłem inną ścieżką. Głownie ze względu na to, że nie chciałem pisać styli do swojego bloga (możliwe, że kiedyś to się zmieni, ale póki co... szybki start).

### Utworzenie repozytorium ###
Spodobał mi się bardzo minimalistyczny styl jakim jest [Pixyll][7]. To responsywny layout do bloga. Sami widzicie na moim blogu jak wygląda. Prosty, przejrzysty, bez fajerwerków, a jednak da się polubić :) Dlatego wszedłem na stronę projektu i w prawym górnym rogu wybrałem Fork.

![GitHub Fork][8]

Potem, wszedłem w ustawienia swojego projektu i zmieniłem jego nazwę zgodnie z konwencją:

```
username.github.io
```

![GitHub settings][9]

### Ustawienia bloga ###
Tutaj w sumie wiele nie trzeba. Jeśli swoje repozytorium nazwałeś zgodnie z konwencją to możesz (a nawet powinieneś) usunąć plik CNAME ze swojego repozytorium. Polecam takze pozaglądać do poszczególnych plików html w głownym katalogu i sprawdzić, czy gdzieś nie warto by było podmienić nazw, które są ewidentnie związane z Twoją nazwą. Pamietaj - w hołdzie twórcom layoutu nigdy nie usuwaj ich ze stopki.
Głowne ustawienia są w pliku _config.yml. Tutaj warto krok po kroku przejrzeć je i odpowiednio je zaktualizować. Ja dodałem komentarze z disqus oraz google analytics. Powłączałem opcje społecznościowe.

### Jak pisać? ###
W tym celu możemy wybrać trzy drogi. Pierwsza opcja to taka, że możemy do katalogu _posts dorzucać swoje posty albo poprzez dodanie pliku bezpośrednio ze strony repozytorium, albo uploadujac wcześniej przygotowany plik również z poziomu strony. Ja wybrałem opcję trzecią. Sklonowałem lokalnie repozytorium poleceniem

```
git clone URL
```

gdzie URL jest adresem Waszego repo. Potem piszę w Lavernie i zapisuję do pliku. Plik wrzucam lokalnie + wszystkie obrazki. Potem jużtylko commit, push i ...

### Kompilacja ###
Jekyll na Githubie robi za nas robotę. Konwertuje wrzucony post do HTML i wysyła do nas raport na maila. Jeśli nic nie zepsuliśmy to na naszej stronie widzimy nowy post.

## Zapamiętaj ##
### Nagłówek posta ###
Każdy post powinien posiadać nagłówek, który jest wymagany przez Jekyll. 
```markdown
---
layout:     post
title:      Blog na GitHubie
date:       2017-02-27 07:00
summary:    Przy ostatnim poście mocno się namęczyłem aby Wam przekazać te wszystkie informacje. Ale jednocześnie dzięki całemu temu czasowi odkryłem fajne sposoby na przyszłość. Oto kilka kroków ku temu aby pisanie bloga stało się o wiele prostsze.
categories: DSP2017 MassCo GitHub Jekyll blog
---
```

Powyższy pochodzi z aktualnego postu. Dzięki temu Jekyll wie jakiego layoutu użyć do renrerowania treści, jaki będzie tytuł i data publikacji. Wyświetli summary na stronie głównej oraz będzie też wyświetlał tagi.

### RSS ###
Rss jest już w tym layoucie zaimplementowany. Ja jednak lekko go zmodyfikowałem bo wrzucał on całego posta do opisu w rss. W pliku feed.xml Powinniśmy zastąpić linijkę description w taki sposób:

```xml
{{ site.description | xml_escape }}
```

Dzięki temu będziemy mieli w RSS skrócony opis wpisu.

## Dziękuję :) ##
Dzięki za uwagę. Mam nadzieję, ze chociaż trochę pomogłem początkującym dev-blogerom. Jakbyście mieli pytania... walcie śmiało!



  [1]: https://daringfireball.net/projects/markdown/syntax
  [2]: http://electron.atom.io/apps/?q=Markdown
  [3]: https://laverna.cc/
  [4]: https://ghost.org/
  [5]: http://logdown.com/
  [6]: https://pages.github.com/
  [7]: http://pixyll.com/
  [8]: {{ site.baseurl }}/images/github-fork.PNG
  [9]: {{ site.baseurl }}/images/github-settings.PNG