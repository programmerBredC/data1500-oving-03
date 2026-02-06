# Besvarelse av refleksjonsspørsmål - DATA1500 Oppgavesett 1.3

Skriv dine svar på refleksjonsspørsmålene fra hver oppgave her.

---

## Oppgave 1: Docker-oppsett og PostgreSQL-tilkobling

### Spørsmål 1: Hva er fordelen med å bruke Docker i stedet for å installere PostgreSQL direkte på maskinen?
Fordelen med Docker er at du kan jobbe i PostgreSQL i en isolert container, som ikke påvirker resten av systemet.



### Spørsmål 2: Hva betyr "persistent volum" i docker-compose.yml? Hvorfor er det viktig?
Persistent volum betyr at det lagres utenfor containeren slik at man kan gjennopptå aktiviteten. 



### Spørsmål 3: Hva skjer når du kjører `docker-compose down`? Mister du dataene?
Med docker-compose down avslutter du gjeldende container. Dataen mistes kun hvis du sletter volumet. 



### Spørsmål 4: Forklar hva som skjer når du kjører `docker-compose up -d` første gang vs. andre gang.
Med docker-compose up -d åpner du en container med et volum. Ved andre gang åpner du et container med det samme volumet. 



### Spørsmål 5: Hvordan ville du delt docker-compose.yml-filen med en annen student? Hvilke sikkerhetshensyn må du ta?
Jeg ville ha sendt docker-compose.yml filen direkte til studenten. 



## Oppgave 2: SQL-spørringer og databaseskjema

### Spørsmål 1: Hva er forskjellen mellom INNER JOIN og LEFT JOIN? Når bruker du hver av dem?
Inner join er default versjonen av Join som returnerer en rad hvis det finnes verdier i begge tabellene. Left Join inkluderer alle radene fra den første tabellen selv om det ikke finnes noen verdi i tabell to. 

### Spørsmål 2: Hvorfor bruker vi fremmednøkler? Hva skjer hvis du prøver å slette et program som har studenter?
Fremmednøkler kobler data mellom tabeller sammen. Dermed kan man ikke uten videre slette data som brukes andre steder. 

### Spørsmål 3: Forklar hva `GROUP BY` gjør og hvorfor det er nødvendig når du bruker aggregatfunksjoner.
Group by grupperer radene i en tabell etter en viss egenskap som gjør at du kan bruke aggregatfunksjoner på deler av tabellen.  


### Spørsmål 4: Hva er en indeks og hvorfor er den viktig for ytelse?
Indeks sørger for at data hentes raskere ved  vise til lokasjonen til dataen.


### Spørsmål 5: Hvordan ville du optimalisert en spørring som er veldig treg?
Jeg hadde sett nærmere på søkealgoritmen. Her kan man gjøre spesifikke endringer som legge til indeks på ofte brukte rader. Man kan også være mer konkret med spørringen ved bruk av WHERE og HAVING slik at utvalget begrenses for hvert søk.  


## Oppgave 3: Brukeradministrasjon og GRANT

### Spørsmål 1: Hva er prinsippet om minste rettighet? Hvorfor er det viktig?
Prinsippet om minste rettighet passer på at brukere kun får tilgang til det de trenger for å gjøre jobben sin. Dermed beskytter man sensitiv data.  

### Spørsmål 2: Hva er forskjellen mellom en bruker og en rolle i PostgreSQL?
En bruker og rolle er i prinsippet det samme, de er begge en rolle. Forskjellen er at en bruker har logg inn rettigheter slik at den kan opererere i databasen. En rolle definerer et set med rettigheter som kan arves av bruker. 

### Spørsmål 3: Hvorfor er det bedre å bruke roller enn å gi rettigheter direkte til brukere?
Ved å gi bruker en rolle slipper man å definere rettigheter per bruker. Det gjør det også enklere å endre rettigheter til en gruppe brukere, og skaper en mer oversiktig struktur i kodingen. 

### Spørsmål 4: Hva skjer hvis du gir en bruker `DROP` rettighet? Hvilke sikkerhetsproblemer kan det skape?
Drop rettigheter gir bruker mulighet til å slette objekter som tabeller fra databasen. Dette kan gjøre en stor mengde data sårbar for sletting eller endring, og kan i sin tur påvirke resten av funksjonen til databasen. 


### Spørsmål 5: Hvordan ville du implementert at en student bare kan se sine egne karakterer, ikke andres?
Jeg ville gitt en tilgang som selekterer bruker sin id. Kanskje kunne man knytte id opp mot radnummer: 
ALTER TABLE karakterer ENABLE ROW LEVEL SECURITY;

CREATE POLICY student_egen_karakter
  ON karakterer
  FOR SELECT
  USING (student_id = current_user);


## Notater og observasjoner

Bruk denne delen til å dokumentere interessante funn, problemer du møtte, eller andre observasjoner:

- Hvordan WHERE og HAVING begge sorterer ut data, men før og etter gruppering.
- Syntaks kan være litt forvirrende med ON, FOR, FROM, USING, og TO. 


## Oppgave 4: Brukeradministrasjon og GRANT

1. **Hva er Row-Level Security og hvorfor er det viktig?**
   - Svar her...

2. **Hva er forskjellen mellom RLS og kolonnebegrenset tilgang?**
   - Svar her...

3. **Hvordan ville du implementert at en student bare kan se karakterer for sitt eget program?**
   - Svar her...

4. **Hva er sikkerhetsproblemene ved å bruke views i stedet for RLS?**
   - Svar her...

5. **Hvordan ville du testet at RLS-policyer fungerer korrekt?**
   - Svar her...

---

## Referanser

- PostgreSQL dokumentasjon: https://www.postgresql.org/docs/
- Docker dokumentasjon: https://docs.docker.com/

