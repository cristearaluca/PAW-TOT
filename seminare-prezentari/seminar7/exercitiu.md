# Exercițiu

Continuăm aplicația de catalog de cărți din laboratorul anterior. Interfața grafică rămâne identică — același `Form1` cu `ListView`, același `FormCarte`. Singura schimbare este în stratul de date: `FakeDatabase` și lista din memorie dispar complet, iar `CarteRepository` este rescris să acceseze o bază de date SQL Server în fișier prin ADO.NET.

Aceasta demonstrează în practică valoarea separării straturilor: înlocuim sursa de date fără să atingem niciun formular.

### Crearea bazei de date

Urmează pașii din documentația de instalare SSDT pentru a crea baza de date.

Creează un fișier de bază de date SQL Server (`.mdf`) în proiect prin **Project → Add → New Item → Service-based Database**, denumit `CartiDB.mdf`.

Creează tabelul `Carti` cu scriptul SQL:

```sql
CREATE TABLE Carti (
    Id          UNIQUEIDENTIFIER PRIMARY KEY,
    Titlu       NVARCHAR(200)   NOT NULL,
    Autor       NVARCHAR(150)   NOT NULL,
    AnAparitie  INT             NOT NULL,
    Gen         NVARCHAR(50)    NOT NULL,
    Pret        DECIMAL(10, 2)  NOT NULL
)
```

Inserează cel puțin trei cărți de test.

### `CarteRepository` cu ADO.NET

Rescrie `CarteRepository` fără nicio referință la `FakeDatabase`. Toate metodele accesează baza de date prin `SqlConnection` și `SqlCommand`.

Declară connection string-ul ca câmp privat constant:

```csharp
private const string CONNECTION_STRING =
    @"Data Source=(LocalDB)\MSSQLLocalDB;AttachDbFilename=|DataDirectory|\CartiDB.mdf;Integrated Security=True";
```

Implementează metodele:

**`GetAll()`** — execută `SELECT ... FROM Carti`, parcurge cu `SqlDataReader`, construiește și returnează o listă de obiecte `Carte`.

**`GetById(int id)`** — execută `SELECT ... FROM Carti WHERE Id = @Id` cu parametru, returnează un singur obiect `Carte` sau `null`.

**`Add(Carte c)`** — execută `INSERT INTO Carti (...) VALUES (...)` cu parametri pentru fiecare coloană.

**`Update(Carte c)`** — execută `UPDATE Carti SET ... WHERE Id = @Id` cu parametri.

**`Delete(int id)`** — execută `DELETE FROM Carti WHERE Id = @Id` cu parametru.

Toate metodele deschid și închid conexiunea în interiorul lor, folosind `using` pentru `SqlConnection`, `SqlCommand` și `SqlDataReader`.

### Integrarea în UI&#x20;

`Form1` și `FormCarte` rămân structural identice cu laboratorul anterior. Singurele modificări necesare:

* `AppDomain.CurrentDomain.SetData("DataDirectory", Application.StartupPath)` adăugat în `Program.cs` înainte de `Application.Run()`, pentru ca `|DataDirectory|` din connection string să fie rezolvat corect

Verifică că toate operațiile CRUD funcționează cu baza de date: adăugarea persistă după repornirea aplicației, editarea și ștergerea sunt reflectate imediat în `ListView`.

### Gestionarea excepțiilor

Împachetează metodele din repository în `try/catch` care prind `SqlException`. La producerea unei erori, aruncă mai departe o `Exception` cu un mesaj clar:

```csharp
catch (SqlException ex)
{
    throw new Exception("Eroare la accesarea bazei de date: " + ex.Message, ex);
}
```

În `Form1`, împachetează apelurile către repository în `try/catch` care prind `Exception` și afișează mesajul cu `MessageBox`.


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-7-ado.net/exercitiu.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
