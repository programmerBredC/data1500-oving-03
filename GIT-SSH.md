# Konfigurasjon av arbeidssmiljø for DATA1500
Bruk god tid for å forstå konfigurasjon og en "prøve-og-feile"-strategi, slik at du sparer tid neste gang du skal øve med Github og Docker. 

Tegnet `$` brukes for å markere prompt og kan variere fra datamaskin til datamaskin (avhenger av lokal konfigurasjon). Eksemplene av utførelse av kommandoene på kommandolinje blir i dette dokumentet fremstilt slikt:
```bash
  $ <kommando>
  <output til stdout og stderr>
```
Tegnet `~` betegner hjemmemappe til den brukerkontoen som man har logget seg inn på datamaskinene med, hvis man bruker Unix/Linux-miljø (Powershell i MS Windows-miljø støtter vanligvis også det; i `cmd` can kommandoen `cd %USERPROFILE%` brukes).

ghbruker - byttes ut med ditt brukernavn for github kontoen, som du bruker i DATA1500;

## macOS 
### Steg 1: Sjekk OpenSSH
Først, sørg for at du har **OpenSSH Client** installert. Moderne versjoner av macOS inkluderer dette som standard. Du kan verifisere installasjonen ved å åpne Terminal og kjøre `ssh -V`.

Eksempel (eksemplene er utført på Sonoma 14.3.1, Apple M2 Pro):

```bash
  $ ssh -V
  OpenSSH_9.4p1, LibreSSL 3.3.6
```
### Steg 2: Generer nøkkel for din konto
Generer et unikt SSH-nøkkelpar for GitHub-kontoen din. Det er kritisk å lagre nøkkel i en fil med et spesifikt navn for å unngå overskriving.

1.  **Åpne Terminal**.

2.  **Generer en nøkkel for din github-konto** (f.eks. `ghbruker`). Bruk `-f` flagget for å spesifisere et unikt filnavn. `ed25519`-algoritmen er anbefalt for bedre sikkerhet og ytelse [1].

    ```bash
    $ ssh-keygen -t ed25519 -C "din-epost-for-ghbruker@example.com" -f ~/.ssh/id_ed25519_ghbruker
    ```

    Alternativet hvis `-f ~/.ssh/id_ed25519_ghbruker` ikke funksjonerer:

    ```bash
    $ ssh-keygen -t ed25519 -C "din-epost-for-ghbruker@example.com"
    ```

    Godkjenne alt uten noen endringer, slik at nøklene blir lagret i mappen `~/.ssh/`, hvor `~` betegner hjemmemappe for den gjeldende brukeren. Endre navnet på nøkkelfilene, slik at det er enklere å vite hvilke nøklder som gjelder hvilken konto (i tilfelle man ønsker å jobbe mot flere kontoer/servere):

    ```bash
    $ cd ~/.ssh
    $ mv id_ed25519 id_ed25519_ghbruker
    $ mv id_ed25519.pub id_ed25519_ghbruker.pub
    ```


    Når du blir spurt om en "passphrase", kan du trykke Enter for å la den være tom, eller angi et passord for ekstra sikkerhet. Du trenger ikke å spesifisere "passphrase" i denne omgangen. 

### Steg 3: Legg til Nøkkelen i SSH-Agenten
SSH-agenten er et bakgrunnsprogram som holder styr på SSH-nøklene dine og passordene deres. Dette forhindrer at du må skrive inn passordet hver gang du kobler til.

1.  **Sørg for at SSH-agenten kjører**. Åpne Terminal som administrator og kjør følgende kommandoer:

Eksempel for å sjekke at ssh-agent utfører:

```bash
    # Sjekk om en prosess med navn ssh-agent utfører på din maskin
    $ ps aux | grep ssh-agent
    janisg            5230   0.0  0.0 407999680   1536   ??  S    Sun09AM   0:00.05 /usr/bin/ssh-agent -l
    janisg           34097   0.0  0.0 408626944   1360 s001  S+    3:02PM   0:00.01 grep ssh-agent
```
Hvis du får lignende output, trenger du ikke å starte agenten på nytt. Hvis du allikevel utfører oppstartskommandoen for ssh-agent, vil du overskrive den kjørende prosessen og den nye prosessen vil ta over. Det "gamle" prosessen vil fortsatte "sove" i prosess-rommet på din datamaskin. Det ser ikke ut at det har store komplikasjoner, men unngå å utføre start-kommandoen, hvis du ser at en prosess allerede utføres.

Eksempel hvis den må startes (tallet for prosessen blir forskjellig fra datamaskin til datamaskin):

```bash
    # Starter agenten
    $ eval "$(ssh-agent -s)"
    Agent pid 8383
```

2.  **Legg til de nye SSH-nøklene** i agenten:

    ```bash
    $ ssh-add ~/.ssh/id_ed25519_ghbruker
    Identity added: /Users/bruker/.ssh/id_ed25519_ghbruker (<din-epost-for-ghbruker>)
    ```
`din-epost-for-ghbruker` bør være den e-posten som er registert i github-kontoen og som man brukte for å generere nøkler i **Steg 2**.

## Steg 4: Legg til Offentlige Nøkler i GitHub

Hver GitHub-konto må kjenne til den offentlige delen av SSH-nøkkelen din for å kunne autentisere deg.

1.  **Kopier innholdet i den offentlige nøkkelen**. For konto `ghbruker`:

    ```powershell
    $ cat ~/.ssh/id_ed25519_ghbruker.pub
    ```
Og kopier outputen fra kommandoen i Terminal til utklippstavle på din datamaskin.

2.  **Naviger til GitHub-innstillingene**:
    - Logg inn på GitHub-kontoen `ghbruker`.
    - Gå til **Settings** > **SSH and GPG keys**.
    - Klikk på **New SSH key**.

3.  **Lim inn nøkkelen**: Gi den et beskrivende navn (f.eks. "Min studie-mac") og lim inn nøkkelen fra utklippstavlen.

## Steg 5: Konfigurer SSH-klienten (`~/.ssh/config`)

Dette er det viktigste steget. Du skal nå fortelle SSH-klienten hvilken nøkkel den skal bruke din github-vert. Dette gjøres i `config`-filen.

1.  **Opprett eller åpne `config`-filen** i `~/.ssh/`-mappen. Du kan bruke en tekst-editor som Notepad eller VS Code.

    ```bash
    # Oppretter filen hvis den ikke finnes
    $ touch ~/.ssh/config

    # Åpner filen i en editor (nano eller vim er editorer, som brukes direkte fra kommandolinje)
    nano ~/.ssh/config
    ```
    Bruk **Ctrl-X** med påfølgende Enter for å lagre innholdet i filen.
    
3.  **Legg til følgende konfigurasjon**. Denne oppretter unike "Host"-alias for hver GitHub-konto.

    ```
    # GitHub-konto 1 (ghbruker)
    Host github.com-ghbruker
        HostName github.com
        User git
        IdentityFile ~/.ssh/id_ed25519_ghbruker
        IdentitiesOnly yes
    ```

| Direktiv       | Beskrivelse                                                                                             |
| :------------- | :------------------------------------------------------------------------------------------------------ |
| `Host`         | Et unikt alias du vil bruke for å referere til denne konfigurasjonen.                                   |
| `HostName`     | Det faktiske vertsnavnet du kobler til (alltid `github.com`).                                           |
| `User`         | Brukernavnet for SSH-tilkoblingen (alltid `git` for GitHub).                                            |
| `IdentityFile` | Stien til den private SSH-nøkkelen som skal brukes for denne verten.                                    |
| `IdentitiesOnly`| `yes` sikrer at SSH kun prøver nøkkelen spesifisert i `IdentityFile`, og ikke alle nøkler i agenten.   |

## Steg 6: Klon og Push med Riktig Konto

Nå som alt er konfigurert, må du bruke de nye `Host`-aliasene i Git-kommandoene dine.

-   **For å klone et repository** fra konto `ghbruker`:

    ```bash
    $ git clone git@github.com-ghbruker:ghbruker/repository-navn.git
    ```
    Hvis URL-en er kopiert med Github `Code`knappen, vil den mangle den spesifikke delen `-ghbruker` etter `git@github.com`.  

-   **For et eksisterende repository**, må du oppdatere `remote`-URL-en:

    ```bash
    # Naviger til repository-mappen
    $ cd sti/til/ditt/repo

    # Oppdater URL-en til å bruke det nye aliaset
    $ git remote set-url origin git@github.com-ghbruker:ghbruker/repository-navn.git
    ```

## MS Windows

Før du begynner, sørg for at du har **OpenSSH Client** installert. Moderne versjoner av Windows 10 og 11 inkluderer dette som standard. Du kan verifisere installasjonen ved å åpne PowerShell og kjøre `ssh -V`.

---

### Steg 1: Generer Unik SSH-Nøkkel

Det første steget er å generere en unik SSH-nøkkel for din GitHub-konto.

1.  **Åpne PowerShell**.

2.  **Generer en nøkkel for din konto** (f.eks. `ghbruker`). Bruk `-f` flagget for å spesifisere et unikt filnavn. Det anbefales å bruke `ed25519`-algoritmen for bedre sikkerhet og ytelse [1]. Vær oppmerksom at prompt-tegnet kan være noe annet enn `$` i Powershell. Det er heller ikke lagt inn output i eksemplene, kun selve kommandoen.

    ```powershell
    $ ssh-keygen -t ed25519 -C "din-epost-for-ghbruker@example.com" -f ~/.ssh/id_ed25519_ghbruker
    ```

3.  **Generer en nøkkel for din andre konto** (f.eks. `brB`), og gi den et annet filnavn.

    ```powershell
    $ ssh-keygen -t ed25519 -C "din-epost-for-ghbruker@example.com" -f ~/.ssh/id_ed25519_ghbruker
    ```

Når du blir spurt om en "passphrase", kan du trykke Enter for å la den være tom, eller angi et passord for ekstra sikkerhet. Trenger ikke å bruke "passphrase" i vårt tilfelle.

### Steg 2: Legg til Nøkkel i SSH-Agenten

SSH-agenten er et bakgrunnsprogram som holder styr på SSH-nøklene dine og passordene deres. Dette forhindrer at du må skrive inn passordet hver gang du kobler til.

1.  **Sørg for at SSH-agenten kjører**. Åpne PowerShell som administrator og kjør følgende kommandoer:

    ```powershell
    # Sjekk statusen til agenten
    $ Get-Service ssh-agent

    # Sett oppstartstypen til automatisk og start tjenesten
    $ Set-Service -Name ssh-agent -StartupType Automatic
    $ Start-Service ssh-agent
    ```

2.  **Legg til den nye SSH-nøkkelen** i agenten:

    ```powershell
    $ ssh-add ~/.ssh/id_ed25519_ghbruker
    
    ```

### Steg 3: Legg til Offentlig Nøkkel i GitHub

En GitHub-konto må kjenne til den offentlige delen av SSH-nøkkelen din for å kunne autentisere deg.

1.  **Kopier innholdet i den offentlige nøkkelen**. For konto `ghbruker`:

    ```powershell
    $ Get-Content ~/.ssh/id_ed25519_ghbruker.pub | Set-Clipboard
    ```

2.  **Naviger til GitHub-innstillingene**:
    - Logg inn på GitHub-kontoen `ghbruker`.
    - Gå til **Settings** > **SSH and GPG keys**.
    - Klikk på **New SSH key**.

3.  **Lim inn nøkkelen**: Gi den et beskrivende navn (f.eks. "Windows Laptop") og lim inn nøkkelen fra utklippstavlen.


### Steg 4: Konfigurer SSH-klienten (`~/.ssh/config`)

Dette er det viktigste steget. Du skal nå fortelle SSH-klienten hvilken nøkkel den skal bruke for hvilken vert. Dette gjøres i `config`-filen.

1.  **Opprett eller åpne `config`-filen** i `~/.ssh/`-mappen. Du kan bruke en tekst-editor som Notepad eller VS Code.

    ```powershell
    # Oppretter filen hvis den ikke finnes
    $ if (-not (Test-Path ~/.ssh/config)) { New-Item ~/.ssh/config }

    # Åpner filen i Notepad (notepad kan ha en del innstillinger med automatisk formattering osv., så det kan oppstå utfordringer med å beholde Markdown-formattering korrekt; spør om hjelp, hvis du opplever problemer med det)
    $ notepad ~/.ssh/config
    ```

2.  **Legg til følgende konfigurasjon**. Denne oppretter unike "Host"-alias for hver GitHub-konto.

    ```
    # GitHub-konto 1 (ghbruker)
    Host github.com-ghbruker
        HostName github.com
        User git
        IdentityFile ~/.ssh/id_ed25519_ghbruker
        IdentitiesOnly yes
    ```

| Direktiv       | Beskrivelse                                                                                             |
| :------------- | :------------------------------------------------------------------------------------------------------ |
| `Host`         | Et unikt alias du vil bruke for å referere til denne konfigurasjonen.                                   |
| `HostName`     | Det faktiske vertsnavnet du kobler til (alltid `github.com`).                                           |
| `User`         | Brukernavnet for SSH-tilkoblingen (alltid `git` for GitHub).                                            |
| `IdentityFile` | Stien til den private SSH-nøkkelen som skal brukes for denne verten.                                    |
| `IdentitiesOnly`| `yes` sikrer at SSH kun prøver nøkkelen spesifisert i `IdentityFile`, og ikke alle nøkler i agenten.   |


### Steg 5: Klon og Push med Riktig Konto

Nå som alt er konfigurert, må du bruke de nye `Host`-aliasene i Git-kommandoene dine.

-   **For å klone et repository** fra konto `brA`:

    ```powershell
    $ git clone git@github.com-ghbruker:ghbruker/repository-navn.git
    ```

-   **For et eksisterende repository**, må du oppdatere `remote`-URL-en:

    ```powershell
    # Naviger til repository-mappen
    $ cd sti/til/ditt/repo

    # Oppdater URL-en til å bruke det nye aliaset
    $ git remote set-url origin git@github.com-ghbruker:ghbruker/repository-navn.git
    ```

### Steg 6: Verifiser Tilkoblingen

Du kan enkelt teste at konfigurasjon fungerer som forventet.

-   **Test tilkobling for `brA`**:

    ```powershell
    ssh -T git@github.com-ghbruker
    ```

    Forventet svar: `Hi brA! You've successfully authenticated, but Github does not provide shell access.`


---


[1] https://en.wikipedia.org/wiki/EdDSA 
