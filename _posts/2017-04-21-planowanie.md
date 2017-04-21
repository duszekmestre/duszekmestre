---
layout:     post
title:      Planowanie
date:       2017-04-21 21:32
summary:    Planowanie to jeden z najważniejszych procesów w każdej pracy. Nie każdy umie to robić. Nie każdy też lubi to robić. Ale mimo wszystko u każdej osoby daje to wymierne korzyści. Jakie? Przekonajcie się sami.
tags:       DSP2017 agile planning
categories: dsp2017
---

Planowanie to jeden z najważniejszych procesów w każdej pracy. Nie każdy umie to robić. Nie każdy też lubi to robić. Ale mimo wszystko u każdej osoby daje to wymierne korzyści. Jakie? Przekonajcie się sami.

## 1. Research ##

Jak dla mnie zawsze 1 krokiem jest research. W tym wypadku nie do końca wiedziałem czego sam oczekuję od swojego projektu, dlatego potrzebowałem przemyśleć wiele kwestii. I zacząłem czerpać inspiracje. Nie znam wszystkich komunikatorów. Ale w swoim życiu spotkałem ich kilka. I jest kilka rzeczy, których mi osobiście brakuje:

1. Stabilności
2. Lekkości
3. Świeżego podejścia

To bardzo takie wysokopoziomowe stwierdzenia. Ale wyznaję zasadę, że od ogółu do szczegółu. Ale gdzie szukać inspiracji? No i generalnie jestem zdania, ze najwięcej inspiracji szukać można u grafików. Dlatego często odwiedzam portal [Behance][1]. Spotkać tam można grafiki wybitnych artystów z całego świata. Ich portfolio i podejście do przeróżnych tematów. Dlatego rozpocząłem swój research właśnie tam.

Po kilkugodzinnym poszukiwaniu odnalazłem kilka dość interesujących prac:

1. [U&Me][2] - zainspirowało mnie do ekranu logowania
2. [Qwirc][3] - przycisk do udostępniania treści
3. [Facebook redesign][4] - ekran aktywności/społeczności
4. [Thrive][5] - planowanie i wyświetlanie wydarzeń
5. [Osome][6] - ogólny wygląd i rama, kolorystyka, czystość

## 2. Funkcjonalności ##

Jak już zdecydowałem co mi z jakiego projektu się podoba to przyszedł czas na rozplanowanie funckjonalności. Wyróżniłem kilka głównych modułów:

1. Aktywności
2. Wyszukiwarka
3. Zespoły
4. Wiadomości
5. Społeczność

Kolejnym krokiem to już głębsze rozkminianie i rozpisanie. Na dziś dzień rozpisałem moduły w następujący sposób:

### Aktywności ###

Krótkie notatki co do funkcjonalności jaką ma pełnić ta część aplikacji:

1. **Upcoming** - nadchodzące wydarzenia
2. **Stars** - wszystkie kanały obserwowane
3. **Mentioned** - Wszystkie wiadomości, w których było się wspomnianym (wywołanym)
4. **Teams news** - zbiorcze aktualności ze wszystkich zespołów (dynamiczna lista z max 5 ostatnimi wiadomościami)
5. **Statuses** - historia stanów użytkowników

Generalnie plan jest taki, aby pokazywać domyślnie wszystko z możliwością odfiltrowania poszczególnych kategorii.

### Zespoły ###

Tutaj troszkę więcej, ponieważ uznałem, że to będzie jeden z ważniejszych modułów w aplikacji. Dlatego najpierw dodawanie (duży przycisk plus):

1. Nazwa
2. Określenie prywatności
	a. **Publiczne** - każdy może znaleźć i dołączyć
	b. **Moderowane** - można szukać i poprosić o dostęp
    c. **Prywatne** - nie można wyszukać, jedynie moderator może dodawać osoby

Następnie sam ekran będzie podzielony na dwie sekcje:

1. **Lista zespołów** - tych, w których użytkownik aktualnie bierze udział.
2. **Wyszukiwanie** - możliwość dołączania do poszczególnych zepsołów i wyszukiwanie po nazwie i opisie

Natomiast ekran już konkretnego zespołu będzie posiadał takie zakładki:

1. Main screen
	a. Możliwość przejścia do czatu zespołu
	b. Nazwa
	c. Data utworzenia
	d. Opis ze wsparciem markdown
	e. Lista uczestników
2. Kalendarz zaplanowanych spotkań i deadlinów
3. Notatki
    a. Możliwość dodawania przez uczestników i akceptacji przez określonych moderatorów
    b. Notatki będą mogły być różnego typu: markdown, file, poll, video, image

### Wiadomości ###

Skoro to komunikator, to nie mogło zabraknąć również samego modułu konwersacji.

1. **Lista konwersacji** - z uczestnikami i zespołami + odfiltrowanie i przeszukiwanie po tytule
2. **Konwersacja**
    a. **Tytuł** - nazwa lub automatyczny tytułz nazwami uczestników
    b. **"i"** - ikona przejścia do informacji o uczestniku lub do karty zepsołu
    c. **"x"** - ikona opuszczenia konwersacji
    d. **Lista wiadomości** - każdy wzbogacony wpis (inny typ niż clear text) powinien mieć możliwość podglądu w prawej częsci ekranu lub na full screen
    e. **mention** - oznaczanie kogoś przez **@**
    f. **tagowanie** - tagi już wkradły się w nasze życie, więc jest to niemal obowiązek aby umozliwić bogate użycie **#**
    g. Współdzielenie bogatszej treści

### Społeczność ###

Cóż - to będzie taka mała kopia Facebooka. Możliwośc udostępniania treści różnorakich publicznie (taka jakby tablica firmowa). Dodatkowo osoby typu moderator będą mogły niektóre wpisy przypinać jako takie ważniejsze.

Jednocześnie ekran ten będzie umożliwiał przeglądanie kontaktów w firmie i dostęp do nich.

## Podsumowanie ##

Cały projekt próćz masowych konwersacji - ma być komunikatorem bardziej dla większych zespołów pracowników. Często w pracy programistycznej spotyka się potrzebę pracy nad określonymi zadaniami (zepsoły) i wymianę treści pomiędzy nimi. Wszyscy w pracy się znają (Społeczność) i dzielą newsami, treściami różnego rodzaju i ogłoszeniami. Ekran aktywności ma być podsumowaniem i pierwszą zakładką, do której bedzie się zaglądać po rozpoczęciu pracy. A same konwersacji to podstawa komunikacji rozproszonej.

## Co dalej? ##

Plan jest. Teraz wielką korzyścią jego jest to, że wiadomo, co trzeba zrobić. Dzięki temu nie zastanawiam ysię przy każdej czynności co jeszcze pozostał odo zrobienia. Oszczędzamy czas. Raz poświęcony owocuje w przyszłości. Oczywiście planować można częściej. Przy każdym module. Można też modyfikować plany na bieżąco. To wszystko jest możliwe. Ale pierwsze planowanie jest ziarnkiem, o które trzeba pielęgnować. Polecam każdemu. A już kolejnym razem rozpocznę pisanie poszczególnych modułów. Dlatego zachęcam do śledzenia kolejnych wspisów. Już w kolejnym będzie o modułach w Angularze oraz o routingu. Zapraszam i do usłyszenia!



  [1]: https://www.behance.net/
  [2]: https://www.behance.net/gallery/28384297/U-Me-Messenger
  [3]: https://www.behance.net/gallery/44416245/Qwirc-Messenger-
  [4]: https://www.behance.net/gallery/48396371/Facebook-Redesign
  [5]: https://www.behance.net/gallery/50691611/Thrive
  [6]: https://www.behance.net/gallery/48490185/Osome-A-new-medical-user-experience