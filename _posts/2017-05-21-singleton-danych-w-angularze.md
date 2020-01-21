---
layout:     post
title:      Singleton danych w angularze?
date:       2017-05-21 16:00
summary:    Czy zawsze musimy odświeżać dane po każdej akcji? A może dąłoby się to zrobić troszkę inaczej i lżej? Sami zobaczcie.
tags:       DSP2017 angular4 service

---

Czy zawsze musimy odświeżać dane po każdej akcji? A może dąłoby się to zrobić troszkę inaczej i lżej? Sami zobaczcie.

## Jedna lista ##

Generalnie dzisiaj nie będzie nic odkrywczego. Jedynie taki powiedzmy good practice. Jeśli mamy np moduł, który zarządza zespołami, to warto trzymać go w jednym stałym miejscu dla całej instancji aplikacji. Takim miejscem jest właśnie dedykowany serwis. Serwisy mają to do siebie, że tworzone są tylko raz. A dzięki temu jeśli to w nich będziemy trzymać dane to te dane będą załadowane jednorazowo. Wyobraźmy sobie, że chodzimy pomiędzy widokami. Jeśli pobieranie listy z API będzie na poziomie komponentów to wtedy będziemy pobierać dane za każdym razem (zmiana routingu to nowa instancja komponentu). A to może obciążyć docelowo nasz serwer! Dlatego warto pobieranie zrobić po stronie serwisu i potem jedynie sprawdzać, czy dane są już pobrane.

### TeamsService ###

Oto i krótki kod:

``` js
import { Injectable } from '@angular/core';
import { Http, Headers, Response, RequestOptionsArgs } from '@angular/http';

import { Observable } from "rxjs/Rx";

import { ApplicationSettings } from "../../../../config/app.settings";

import { AppAuthorization } from '../../../../core/app.authorization'

import { Team } from "./models/team";
import { CreateTeamVM } from "./models/models";



@Injectable()
export class TeamsService {
    teams: Array;
    teamsLoaded: boolean;

    API_URL: string;

    constructor(private http: Http,
        private AppAuthorization: AppAuthorization) {
        this.API_URL = ApplicationSettings.apiURL;
        this.teamsLoaded = false;
    }

    loadTeams() {
        let url = this.API_URL + "/Teams";

        var headers = this.AppAuthorization.createAuthorizationHeader(null);

        return this.http.get(url, {
            headers: headers
        })
            .map(response => response.json())
            .do(data => {
                this.teamsLoaded = true;
                this.teams = data;                
            })            
            .catch(error => {
                return Observable.throw(error);
            });
    }

    createTeam(team: CreateTeamVM) {
        let url = this.API_URL + "/Teams";

        var headers = this.AppAuthorization.createAuthorizationHeader(null);

        return this.http.post(url, team, {
            headers: headers
        })
            .map(response => response.json())
            .do(data => {
                let newTeam = new Team();
                newTeam.name = team.name;
                newTeam.id = data;

                this.teams.push(newTeam);                
            })            
            .catch(error => {
                return Observable.throw(error);
            });
    }
}
```

Możecie zauważyć, że pobieranie zespołów powoduje natychmiastowo także ich zapisanie do lokalnej zmiennej i ustawienie flagi **teamsLoaded**. Podobnie jest z dodawaniem nowego zespołu. Nie musimy po takiej operacji przeładowywać całej listy a jedynie dodajemy tam jeden element i lista nadal jest w aktualnym stanie. A jak wygląda to po stronie widoku?

### My teams component ###

```js
import { Component, OnInit } from '@angular/core';

import { TeamsService } from "../../logic/teams.service"
import { Team } from "../../logic/models/team";

@Component({
  templateUrl: './my-teams.html'
})
export class MyTeamsComponent implements OnInit {
  teams: Array;

  constructor(private teamsService: TeamsService) {
  }


  ngOnInit() {
    if (this.teamsService.teamsLoaded) {
      this.teams = this.teamsService.teams;
      return;
    }

    this.teamsService.loadTeams()
      .subscribe((data) => {
        this.teams = this.teamsService.teams;
      }, (error) => {
        console.error(error);
      });
  }
}
```

Krótko, prawda? Na metodzie **ngOnInit**, która wywołuje się po załadowaniu komponentu sprawdzamy najpierw, czy dane są już załadowane. Jeśli tak - pobieramy statyczną kolekcję. Jeśli nie - wywołujemy ładowanie danych i bierzemy już kolekcję gotową. Proste, prawda?

## Podsumowanie ##

Taki zabieg nie jest trudny. Ale dzięki niemu operujemy na raz pobranych danych i nie musimy zabijać naszego serwera.

## Co dalej? ##

Generalnie **powoli** piszę kolejne kroki aplikacji. Niestety nawet bardzo powoli. Ale mimo wszystko idzie do przodu. Dosć duży projekt - a czasu coraz mniej. Ładna pogoda zachęca bardziej do spędzania czasu na powietrzu niż przed komputerem. Ale staram się przynajmniej raz w tygodniu na trochę do tego usiąść i dopisać kolejny klocek do całej budowli. 

A tymczasem do usłyszenia!