# Rezumat

### 1. Ce este ADO.NET și cum se conectează la SQL Server

ADO.NET este biblioteca .NET pentru acces la baze de date relaționale — explicit și manual. Tu scrii SQL, tu deschizi conexiunile, tu citești rezultatele.

```
SqlConnection → SqlCommand → SqlDataReader
    "Usa"      "Ce intrebi"   "Rezultatele"
```

| Concept                      | Ce înseamnă                                                 |
| ---------------------------- | ----------------------------------------------------------- |
| **`SqlConnection`**          | Conexiunea fizică — trebuie deschisă și închisă             |
| **`SqlCommand`**             | Comanda SQL de executat                                     |
| **`SqlDataReader`**          | Cursorul pentru citirea rândurilor unui `SELECT`            |
| **Connection string**        | Adresa și opțiunile de conectare                            |
| **`(LocalDB)\MSSQLLocalDB`** | Instanța SQL Server ușoară instalată cu Visual Studio       |
| **`\|DataDirectory\|`**      | Înlocuit cu `Application.StartupPath` setat în `Program.cs` |
| **`using` pentru conexiuni** | Garantează închiderea indiferent de excepții                |

```csharp
// Program.cs — obligatoriu pentru rezolvarea |DataDirectory|
AppDomain.CurrentDomain.SetData("DataDirectory", Application.StartupPath);
```

### 2. Instalarea SSDT și crearea bazei de date

| Pas                  | Unde                                                                                        |
| -------------------- | ------------------------------------------------------------------------------------------- |
| Instalare SSDT       | Visual Studio Installer → Modify → Individual Components → „SQL Server Data Tools"          |
| Creare fișier `.mdf` | Project → Add → New Item → Data → Service-based Database                                    |
| Creare tabel         | Server Explorer → click dreapta conexiune → New Query → `CREATE TABLE`                      |
| Inserare date test   | Query Editor → `INSERT INTO` → Execute                                                      |
| Copiere la build     | Solution Explorer → `CartiDB.mdf` → Properties → Copy to Output Directory = „Copy if newer" |

```sql
CREATE TABLE Carti (
    Id         UNIQUEIDENTIFIER PRIMARY KEY,
    Titlu      NVARCHAR(200) NOT NULL,
    Autor      NVARCHAR(150) NOT NULL,
    AnAparitie INT           NOT NULL,
    Gen        NVARCHAR(50)  NOT NULL,
    Pret       DECIMAL(10,2) NOT NULL
)
```

### 3. `SqlCommand` și `ExecuteNonQuery`

```csharp
// INSERT
using (SqlCommand cmd = new SqlCommand(
    "INSERT INTO Carti (Titlu, Autor, ...) VALUES (@Titlu, @Autor, ...)", conn))
{
    cmd.Parameters.AddWithValue("@Titlu", c.Titlu);
    cmd.Parameters.AddWithValue("@Autor", c.Autor);
    // ...
    cmd.ExecuteNonQuery();
}

// UPDATE
"UPDATE Carti SET Titlu = @Titlu, ... WHERE Id = @Id"

// DELETE
"DELETE FROM Carti WHERE Id = @Id"
```

| Concept                       | Ce înseamnă                                                           |
| ----------------------------- | --------------------------------------------------------------------- |
| **`ExecuteNonQuery()`**       | Execută `INSERT`/`UPDATE`/`DELETE` — returnează nr. rânduri afectate  |
| **Parametri `@Nume`**         | Previn SQL Injection — NICIODATĂ concatenare de string-uri            |
| **`Parameters.AddWithValue`** | Adaugă parametru inferând tipul din valoarea .NET                     |
| **`WHERE Id = @Id`**          | Obligatoriu la `UPDATE` și `DELETE` — altfel afectezi toate rândurile |
| **`SCOPE_IDENTITY()`**        | Returnează Id-ul generat după `INSERT` — citit cu `ExecuteScalar()`   |

### 4. `SqlDataReader`

```csharp
using (SqlCommand cmd = new SqlCommand("SELECT Id, Titlu, Autor, AnAparitie, Gen FROM Carti", conn))
using (SqlDataReader reader = cmd.ExecuteReader())
{
    while (reader.Read())  // GetAll — parcurge toate randurile
    {
        Carte c = new Carte();
        c.Id         = (Guid)reader["Id"];
        c.Titlu      = reader["Titlu"].ToString();
        c.Autor      = reader["Autor"].ToString();
        c.AnAparitie = (int)reader["AnAparitie"];
        c.Gen        = (GenCarte)Enum.Parse(typeof(GenCarte), reader["Gen"].ToString());
        carti.Add(c);
    }
}

// GetById — un singur rand
if (reader.Read()) { ... return carte; }
return null;
```

| Concept                     | Ce înseamnă                                                |
| --------------------------- | ---------------------------------------------------------- |
| **`ExecuteReader()`**       | Execută `SELECT` și returnează reader                      |
| **`reader.Read()`**         | Avansează la următorul rând, returnează `true` dacă există |
| **`reader["NumeColoana"]`** | Valoarea coloanei ca `object` — necesită cast              |
| **`Enum.Parse`**            | Convertește string-ul din BD înapoi la valoarea enum       |

### 5. `CarteRepository` complet cu ADO.NET

```csharp
class CarteRepository
{
    private const string CONNECTION_STRING =
        @"Data Source=(LocalDB)\MSSQLLocalDB;" +
        @"AttachDbFilename=|DataDirectory|\CartiDB.mdf;Integrated Security=True";

    private Carte CitesteCarte(SqlDataReader reader) { ... }  // helper privat

    public List<Carte> GetAll()   { /* SELECT + while reader.Read() */ }
    public Carte GetById(Guid id)  { /* SELECT WHERE Id + if reader.Read() */ }
    public void Add(Carte c)      { /* INSERT + SCOPE_IDENTITY() */ }
    public void Update(Carte c)   { /* UPDATE WHERE Id */ }
    public void Delete(Guid id)    { /* DELETE WHERE Id */ }
}
```

| Aspect                        | Lab 05                                         | Lab 06                        |
| ----------------------------- | ---------------------------------------------- | ----------------------------- |
| **Sursa de date**             | `FakeDatabase.Carti` (static)                  | SQL Server `.mdf`             |
| **`Id` generat**              | Contor static în `Carte`                       | `IDENTITY(1,1)` în SQL Server |
| **Persistența datelor**       | Pierdute la închidere                          | Păstrate permanent            |
| **`Form1`, `FormCarte`**      | —                                              | Neschimbate                   |
| **Interfața repository-ului** | `GetAll`, `GetById`, `Add`, `Update`, `Delete` | Identică                      |

Gestionarea excepțiilor: repository prinde `SqlException`, aruncă `Exception` cu mesaj clar. `Form1` prinde `Exception` și afișează cu `MessageBox`.


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-7-ado.net/rezumat.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
