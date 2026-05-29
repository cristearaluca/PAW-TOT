# De ce SELECT \* este periculos

`SELECT *` înseamnă „returnează toate coloanele din tabel". Este comod de scris și funcționează corect atâta timp cât structura tabelului nu se schimbă. Problema apare exact când structura se schimbă — și în orice aplicație reală, structura se schimbă.

### Coloane noi în tabel sparg codul existent

Imaginează-ți că după câteva săptămâni adaugi o coloană nouă în tabelul `Carti` — de exemplu `DataAdaugarii DATETIME`. Dacă folosești `SELECT *`, această coloană apare automat în rezultatul interogării, chiar dacă codul C# nu știe nimic de ea.

În sine aceasta nu produce o eroare imediat. Dar dacă ai cod care accesează coloanele după index (`reader.GetString(5)` în loc de `reader["Gen"]`), toți indecșii de după coloana nou inserată sunt acum decalați cu unu și returnează date greșite — fără niciun mesaj de eroare.

Chiar și cu acces după nume, coloana suplimentară consumă rețea și memorie inutil, pentru că citești date pe care nu le folosești.

### Performanță și trafic inutil

`SELECT *` returnează toate coloanele, inclusiv cele de care nu ai nevoie. Într-o aplicație reală, un tabel poate avea zeci de coloane — texte lungi, date binare, câmpuri de audit. Dacă ai nevoie doar de `Id` și `Titlu` pentru a popula o listă, returnarea tuturor celorlalte coloane înseamnă trafic de rețea inutil între aplicație și baza de date și memorie consumată pentru date care sunt ignorate imediat.

### Ambiguitate la JOIN-uri

Când interogezi mai multe tabele cu `JOIN`, `SELECT *` returnează toate coloanele din toate tabelele. Dacă două tabele au o coloană cu același nume — de exemplu, ambele au `Id` — rezultatul conține două coloane numite `Id` și accesul prin `reader["Id"]` devine ambiguu sau returnează valoarea din primul tabel întâlnit, în funcție de driver.

```sql
-- Ambiguu: care Id se returneaza?
SELECT * FROM Carti
JOIN Autori ON Carti.AutorId = Autori.Id

-- Explicit si clar
SELECT Carti.Id, Carti.Titlu, Autori.Nume
FROM Carti
JOIN Autori ON Carti.AutorId = Autori.Id
```

### Varianta corectă: coloane explicite

Specifică întotdeauna exact coloanele de care ai nevoie:

```csharp
// In loc de:
using (SqlCommand cmd = new SqlCommand("SELECT * FROM Carti", conn))

// Foloseste:
using (SqlCommand cmd = new SqlCommand(
    "SELECT Id, Titlu, Autor, AnAparitie, Gen, Pret FROM Carti", conn))
```

Avantajele sunt concrete:

**Contractul este explicit.** Codul C# și schema SQL sunt sincronizate vizibil. Oricine citește interogarea știe exact ce date se transferă.

**Erorile sunt detectate mai devreme.** Dacă redenumești o coloană în SQL și uiți să actualizezi interogarea, aceasta eșuează imediat cu un mesaj clar — nu silent și la runtime.

**Performanța este mai bună.** SQL Server poate optimiza mai bine o interogare cu coloane explicite. Nu se transferă date inutile prin rețea.

**JOIN-urile sunt sigure.** Nu există ambiguitate când mai multe tabele au coloane cu același nume.

### Când `SELECT *` este acceptabil

Există un singur context în care `SELECT *` este uzual și acceptat: interogările ad-hoc scrise manual în Query Editor pentru a inspecta datele în timpul dezvoltării. Când vrei să verifici rapid ce este în tabel, `SELECT * FROM Carti` este perfect rezonabil.

Nu este acceptabil în codul aplicației — în metodele repository-ului, în stored procedures sau oriunde rezultatul este procesat programatic.


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-7-ado.net/de-ce-select-este-periculos.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
