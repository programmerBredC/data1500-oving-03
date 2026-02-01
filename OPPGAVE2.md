# Oppgave 2: SQL-spørringer og databaseskjema

## Læringsmål

Etter å ha fullført denne oppgaven skal du:
- Forstå relasjonsdatabaseskjemaet
- Skrive SELECT-spørringer med WHERE, JOIN, ORDER BY
- Bruke aggregatfunksjoner (COUNT, AVG, SUM)
- Forstå primærnøkler, fremmednøkler og integritetskontroller
- Kunne lese og tolke databasediagrammer

## Bakgrunn

En relasjonsdatabase er organisert i **tabeller** som består av **rader** og **kolonner**. Tabeller er knyttet sammen gjennom **fremmednøkler**.

**Databaseskjemaet for DATA1500:**

```
programmer (program_id, program_navn, beskrivelse, opprettet)                                         
studenter (student_id, fornavn, etternavn, epost, program_id, opprettet)              
emneregistreringer (registrering_id, student_id, emne_id, semester, karakter, registrert_dato)
emner (emne_id, emne_kode, emne_navn, studiepoeng, beskrivelse, opprettet)
```

**Nøkkelbegreper:**
- **Primærnøkkel (PK):** Unik identifikator for hver rad (f.eks. `student_id`)
- **Fremmednøkkel (FK):** Referanse til primærnøkkel i en annen tabell
- **Integritetskontroll:** Regler som sikrer datakonsistens (f.eks. `CHECK (studiepoeng > 0)`)

## Oppgave

Alle spørringene skal kjøres i PostgreSQL. Du kan bruke `psql` (klienten for postgres-serveren) eller et GUI-verktøy som pgAdmin eller DBeaver. **Anbefalt** å bruke `psql` først og så gå over til GUI-verktøy.

For å bruke `psql` kan du "logge inn" i container og i psql-shell med følgende kommandoer:
```bash
    $ docker-compose up 
    $ docker-compose exec postgres psql -U admin -d data1500_db
```

Forventet output er en psql-shell:
```bash
    psql (15.15)
    Type "help" for help.

    data1500_db=#
```

Utforsk "help". I `psql` finnes det egne kommandoer (ikke SQL-basert) for å få ut metadata om database. For eksempel, vil kommandoen `\d` liste ut alle tabeller, views og sekvenser i den gjeldende databasen `data1500_db`.

### Del 1: Grunnleggende SELECT-spørringer

**1.1** Hent alle studenter med fornavn, etternavn og epost:

```sql
SELECT fornavn, etternavn, epost FROM studenter;
```

**1.2** Hent alle emner sortert etter emne_navn:

```sql
SELECT emne_kode, emne_navn, studiepoeng FROM emner ORDER BY emne_navn;
```

**1.3** Hent alle studenter fra Informatikk-programmet (program_id = 1):

```sql
SELECT fornavn, etternavn, epost FROM studenter WHERE program_id = 1;
```

### Del 2: JOIN-spørringer

**2.1** Hent alle studenter med deres program:

```sql
SELECT 
    s.fornavn, 
    s.etternavn, 
    p.program_navn
FROM studenter s
LEFT JOIN programmer p ON s.program_id = p.program_id;
```

**2.2** Hent alle emneregistreringer med studentnavn og emnenavn:

```sql
SELECT 
    s.fornavn,
    s.etternavn,
    e.emne_navn,
    er.karakter,
    er.semester
FROM emneregistreringer er
JOIN studenter s ON er.student_id = s.student_id
JOIN emner e ON er.emne_id = e.emne_id
ORDER BY s.etternavn, e.emne_navn;
```

**2.3** Hent alle emner som DATA1500-studenter er registrert på:

```sql
SELECT DISTINCT e.emne_kode, e.emne_navn
FROM emneregistreringer er
JOIN emner e ON er.emne_id = e.emne_id
WHERE er.student_id IN (
    SELECT student_id FROM studenter WHERE program_id = 1
);
```

### Del 3: Aggregatfunksjoner

**3.1** Tell antall studenter per program:

```sql
SELECT 
    p.program_navn,
    COUNT(s.student_id) as antall_studenter
FROM programmer p
LEFT JOIN studenter s ON p.program_id = s.program_id
GROUP BY p.program_id, p.program_navn
ORDER BY antall_studenter DESC;
```

**3.2** Hent gjennomsnittlig karakter per emne:

```sql
SELECT 
    e.emne_navn,
    AVG(CAST(SUBSTRING(er.karakter, 1, 1) AS INT)) as gjennomsnitt
FROM emneregistreringer er
JOIN emner e ON er.emne_id = e.emne_id
WHERE er.karakter IS NOT NULL
GROUP BY e.emne_id, e.emne_navn;
```

**3.3** Hent studenter som har flere enn 1 emneregistrering:

```sql
SELECT 
    s.fornavn,
    s.etternavn,
    COUNT(er.registrering_id) as antall_emner
FROM studenter s
LEFT JOIN emneregistreringer er ON s.student_id = er.student_id
GROUP BY s.student_id, s.fornavn, s.etternavn
HAVING COUNT(er.registrering_id) > 1
ORDER BY antall_emner DESC;
```

### Del 4: Databaseskjema-analyse

**4.1** Hent alle fremmednøkler i databasen:

```sql
SELECT 
    tc.table_name, 
    kcu.column_name, 
    ccu.table_name AS foreign_table_name,
    ccu.column_name AS foreign_column_name
FROM information_schema.table_constraints AS tc
JOIN information_schema.key_column_usage AS kcu
    ON tc.constraint_name = kcu.constraint_name
JOIN information_schema.constraint_column_usage AS ccu
    ON ccu.constraint_name = tc.constraint_name
WHERE tc.constraint_type = 'FOREIGN KEY';
```

**4.2** Hent alle indekser:

```sql
SELECT 
    schemaname,
    tablename,
    indexname
FROM pg_indexes
WHERE schemaname = 'public'
ORDER BY tablename, indexname;
```

## Oppgaver du skal løse

Skriv SQL-spørringer som besvarer følgende spørsmål:

1. **Hent alle studenter som ikke har noen emneregistreringer**
2. **Hent alle emner som ingen studenter er registrert på**
3. **Hent studentene med høyeste karakter per emne**
4. **Lag en rapport som viser hver student, deres program, og antall emner de er registrert på**
5. **Hent alle studenter som er registrert på både DATA1500 og DATA1100**

1. SELECT fornavn, etternavn, epost FROM studenter WHERE program_id IS NULL;
2. SELECT DISTINCT e.emne_navn, COUNT (er.student_id) FROM emner e LEFT JOIN emneregistreringer er ON er.emne_id =
   e.emne_id GROUP BY e.emne_navn HAVING COUNT(er.student_id) = 0

**Viktig:** Lagre alle spørringene dine i en fil `oppgave2_losning.sql` i mappen `test-scripts` for at man kan teste disse med kommando:

```bash
docker-compose exec postgres psql -U admin -d data1500_db -f test-scripts/oppgave2_losning.sql
```

Du kan selv bruke denne kommandoen for å teste dine SQL-spørringer og SQL-setnigner. 

## Refleksjonsspørsmål

Besvar refleksjonsspørsmål i filen **besvarelse-refleksjon.md**


## Avslutning

Når du er ferdig:
- Du forstår (overfladisk) relasjonsdatabaseskjemaet
- Du kan skrive noen SELECT-spørringer (forventes ikke en detaljert kunnskap nå)
- Du er klar for oppgave 3 (brukeradministrasjon)
