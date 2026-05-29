# SQL Server Data Tools

**SQL Server Data Tools** (SSDT) este o extensie pentru Visual Studio care adaugă suport complet pentru lucrul cu baze de date SQL Server direct din IDE. Fără SSDT, Visual Studio nu poate crea fișiere `.mdf`, nu poate afișa structura tabelelor în Server Explorer și nu oferă editorul vizual de tabele.

SSDT instalează câteva componente esențiale:

* **Server Explorer** cu suport pentru conexiuni SQL Server
* **Table Designer** — editorul vizual pentru structura tabelelor
* **Query Editor** — editor SQL cu syntax highlighting și execuție directă
* **LocalDB** — instanța ușoară de SQL Server pentru dezvoltare locală

### Instalarea SSDT din Visual Studio Installer

SSDT nu este instalat implicit cu Visual Studio — trebuie adăugat separat.

**Pasul 1:** Deschide **Visual Studio Installer** din meniul Start

<figure><img src="/files/xGcyyh6BO5jvGQ5aaBpg" alt=""><figcaption></figcaption></figure>

**Pasul 2:** În Visual Studio Installer, click pe **Modify** pentru instalarea ta de Visual Studio.

<figure><img src="/files/Fzx339Cv09wF7RHoc1GO" alt=""><figcaption></figcaption></figure>

**Pasul 3:** În tab-ul **Individual components**, caută „SQL Server Data Tools". Bifează componenta și apasă **Modify**.

<figure><img src="/files/lkN7AcbfoD9Sgj2PRNxr" alt=""><figcaption></figcaption></figure>

**Pasul 4:** Repornește Visual Studio după finalizarea instalării.

### Crearea fișierului de bază de date în proiect

Odată SSDT instalat, adaugi baza de date direct în proiectul Visual Studio:

**Pasul 1:** Click dreapta pe proiect în Solution Explorer → **Add → New Item**.

**Pasul 2:** În fereastra de adăugare, caută sau navighează la categoria **Data** și selectează **Service-based Database**. Denumește fișierul `CartiDB.mdf`.

<figure><img src="/files/ChYUZMBeb4P0Z6IUPcBC" alt=""><figcaption></figcaption></figure>

**Pasul 3:** Click **Add**. Visual Studio creează fișierul `CartiDB.mdf` în proiect și îl adaugă automat în Solution Explorer. Se deschide și **Server Explorer** cu conexiunea la baza de date nouă. În cazul în care nu se deschide automat, Server Explorer poate fi găsit în meniul **View → Server Explorer**

<figure><img src="/files/yaLcb7OliShSF1IBk9i4" alt=""><figcaption></figcaption></figure>

### Crearea tabelului `Carti`

Cu baza de date creată, adăugăm tabelul prin Query Editor:

**Pasul 1:** În Server Explorer, click dreapta pe conexiunea la `CartiDB.mdf` → **New Query**.

<figure><img src="/files/aQ069drfyYS7EkyaJrhG" alt=""><figcaption></figcaption></figure>

**Pasul 2:** În Query Editor, scrie și execută scriptul de creare:

```sql
CREATE TABLE Carti (
    Id          UNIQUEIDENTIFIER PRIMARY KEY,
    Titlu       NVARCHAR(200)   NOT NULL,
    Autor       NVARCHAR(150)   NOT NULL,
    AnAparitie  INT             NOT NULL,
    Gen         NVARCHAR(50)    NOT NULL
)
```

**Pasul 3:** Apasă butonul **Execute** (sau `Ctrl+Shift+E`) pentru a rula scriptul.

<figure><img src="/files/n3sNrBmZBskXHdrp4d71" alt=""><figcaption></figcaption></figure>

**Câteva detalii despre structura tabelului:**

`UNIQUEIDENTIFIER` este tipul de date pentru un Guid — un identificator unic de 128 de biți, generat în cod cu `Guid.NewGuid()` și transmis explicit la `INSERT`. Spre deosebire de `INT IDENTITY`, SQL Server nu generează valoarea — aceasta vine din aplicație.

`NVARCHAR` este tipul de date pentru text Unicode. Litera `N` înseamnă că suportă caractere din orice alfabet, inclusiv diacritice românești. Numărul din paranteză este lungimea maximă în caractere.

`NOT NULL` înseamnă că coloana nu acceptă valori lipsă. Orice `INSERT` care omite o coloană `NOT NULL` eșuează cu eroare.

**Inserarea datelor de test:**

Deoarece `Id`-ul este generat în cod, la inserarea datelor de test trebuie să furnizezi Guid-uri explicite:

```sql
INSERT INTO Carti (Id, Titlu, Autor, AnAparitie, Gen) VALUES
(NEWID(), N'Mândrie și prejudecată', N'Jane Austen', 1813, N'Roman'),
(NEWID(), N'Război și pace', N'Lev Tolstoi', 1869, N'Roman'),
(NEWID(), N'Crimă și pedeapsă', N'Feodor Dostoievski', 1866, N'Roman'),
(NEWID(), N'Don Quijote', N'Miguel de Cervantes', 1605, N'Roman'),
(NEWID(), N'Marele Gatsby', N'F. Scott Fitzgerald', 1925, N'Roman'),
(NEWID(), N'Originea speciilor', N'Charles Darwin', 1859, N'Stiinta'),
(NEWID(), N'Scurtă istorie a timpului', N'Stephen Hawking', 1988, N'Stiinta'),
(NEWID(), N'Dune', N'Frank Herbert', 1965, N'Fictiune'),
(NEWID(), N'1984', N'George Orwell', 1949, N'Fictiune'),
(NEWID(), N'Viața mea', N'Charlie Chaplin', 1964, N'Biografie'),
(NEWID(), N'Clean Code', N'Robert C. Martin', 2008, N'Tehnic'),
(NEWID(), N'Design Patterns', N'Erich Gamma', 1994, N'Tehnic'),
(NEWID(), N'Povestiri zen', N'Nyogen Senzaki', 1939, N'Altele'),
(NEWID(), N'Cartea curiozităților', N'National Geographic', 2012, N'Altele')
```

`NEWID()` este funcția SQL Server care generează un Guid nou — echivalentul lui `Guid.NewGuid()` în C#. O folosim doar pentru datele de test inserate manual. În aplicație, Guid-ul este generat în constructorul lui `Carte` și trimis ca parametru.

### Vizualizarea datelor din tabel

Pentru a verifica că tabelul și datele există corect, poți rula un `SELECT` simplu:

```sql
SELECT * FROM Carti
```

<figure><img src="/files/OQt8o0lnuWPXgNg5xrLd" alt=""><figcaption></figcaption></figure>

### Proprietatea `Copy to Output Directory`

Fișierul `CartiDB.mdf` este inclus în proiect, dar trebuie copiat și în folderul `bin\Debug` unde rulează aplicația, altfel connection string-ul nu îl găsește.

Click pe `CartiDB.mdf` în Solution Explorer, deschide fereastra **Properties** (`F4`) și setează proprietatea **Copy to Output Directory** la **Copy if newer**.

<figure><img src="/files/GHJLG1sD3qI6bcSI4prQ" alt=""><figcaption></figcaption></figure>

**Copy if newer** înseamnă că fișierul este copiat în `bin\Debug` la build numai dacă versiunea din proiect este mai nouă decât cea deja copiată. Aceasta evită suprascrierea datelor introduse în timp ce aplicația rulează.


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-7-ado.net/sql-server-data-tools.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
