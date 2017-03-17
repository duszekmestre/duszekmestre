---
layout:     post
title:      Token based authentication - cz. 1
date:       2017-03-17 20:00
summary:    Dzisiaj rozpoczynam pierwszą część, która jest jednocześnie wprowadzeniem do autentykacji na tokenach.
tags:       DSP2017 token asp.net authentication
categories: dsp2017
---

Dzisiaj rozpoczynam pierwszą część, która jest jednocześnie wprowadzeniem do autentykacji na tokenach.

## Token - a co to? ##

Cóż. To taki ciąg znaków, który jest wygenerowany na okres sesji logowania. Uzyskujemy go podczas logowania i wykorzystujemy aż do jego wygaśnięcia lub zakończenia sesji użytkownika. Tokeny są następcami ciasteczek (aczkolwiek bardzo często sam token jest przechowywany w ciasteczku). 

Największy hype na tokeny zrodził się w miarę rosnącej siły frameworków javascriptowych i konsumpcji API. Dlaczego akurat tokeny?

### Skalowalność ###

No tutaj troszkę pojechałem. Tokeny ie dają skalowalności. Ale dobrze napisane repozytorium tokenów jest odporne na skalowalność. Teoretycznie token jest ... **"self contained"**. Czyli sam zawiera w sobie informacje. Nie do końca się z tym zgadzam. Jak dla mnie token o wiele bardziej jest jakby wskaźnikiem na użytkownika. A skoro serwery podczas skalowania chcą korzystać z dancyh użytkownika to najlepiej im dać informację o jakiego użytkownika nam chodzi. I to daje nam token.

### Niezależność ###

Tak to sobie nazwałem. Generalnei aplikacja mobilna/webowa/desktop, która korzysta z API wymaga na początek jednorazowej autentykacji. Wtedy serwer (API) zwraca wygenerowany przez siebie token i od tej pory aplikacja kliencka nie musi się już niczym martwić prócz zapamiętania tokenu.

### Przyjaciel mobilności ###

Głownie chodzi o ciasteczka. O ile pisze się klienta bazującego na przeglądarce (strona WWW, Electron) to ciasteczka są dostępne. Ale ciasteczka w aplikacjach mobilnych? To już nie tędy droga. Dlatego token można przesłać jako odpowiedź z autentykacji i go przechować. To znacznie prostsza droga.

## Plany ##

Już o tokenach troszkę wiemy. Znamy zalety. Ale co dalej? Oto kilka rzeczy, które będą opisane w najbliższej serii:

1. Logowanie i rejestracja użytkownika (dość nietypowa)
2. Zabezpieczenie zasobów przed niepowołanym dostępem

Technologia? Tutaj główną rolę spełni ASP.NET Web API w wersji 2. A co głębiej. Przekonacie się sami :)

## Do usłyszenia! ##

Trzymajcie się i do poczytania wkrótce!