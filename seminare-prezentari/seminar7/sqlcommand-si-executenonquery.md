# SqlCommand și ExecuteNonQuery

Operațiile `INSERT`, `UPDATE` și `DELETE` modifică date în baza de date, dar nu returnează rânduri de date. Ele returnează doar un număr întreg, și anume câte rânduri au fost afectate. Pentru aceste operații folosim metoda `ExecuteNonQuery()`.

```csharp
int randuriAfectate = cmd.ExecuteNonQuery();
// randuriAfectate = 1 daca un rand a fost modificat/sters
// randuriAfectate = 0 daca niciun rand nu a indeplinit conditia WHERE
```

Valoarea returnată de `ExecuteNonQuery()` este utilă pentru a detecta situații neașteptate — de exemplu, dacă un `DELETE` cu `WHERE Id = @Id` returnează 0, înseamnă că nu a existat niciun element cu acel id.

### Parametrii SQL

Când comanda SQL folosește valori primite de la utilizator sau din cod, există tentația de a construi string-ul SQL prin concatenare:

```csharp
// NICIODATA nu face asa ceva
string sql = "INSERT INTO Carti (Titlu) VALUES ('" + titlu + "')";
```

Aceasta este o vulnerabilitate gravă numită **SQL Injection**. Dacă `titlu` conține textul `'); DROP TABLE Carti; --`, comanda construită devine:

```sql
INSERT INTO Carti (Titlu) VALUES (''); DROP TABLE Carti; --')
```

SQL Server execută ambele instrucțiuni și tabelul este șters. Chiar și fără intenție malițioasă, un apostrof simplu în titlul unei cărți ar cauza eroare de sintaxă SQL.

Soluția este să folosești întotdeauna **parametri**. Parametrii sunt marcatori de tip `@NumeParametru` în textul SQL, cărora le atribui valori separat. SQL Server tratează valorile parametrilor ca date pure, niciodată ca parte din sintaxa SQL:

```csharp
// CORECT — valoarea titlului nu poate modifica niciodata sintaxa SQL
using (SqlCommand cmd = new SqlCommand(
    "INSERT INTO Carti (Titlu, Autor) VALUES (@Titlu, @Autor)", conn))
{
    cmd.Parameters.AddWithValue("@Titlu", carte.Titlu);
    cmd.Parameters.AddWithValue("@Autor", carte.Autor);
    cmd.ExecuteNonQuery();
}
```

Chiar dacă `carte.Titlu` conține `'); DROP TABLE Carti; --`, SQL Server îl tratează ca text literal, nu ca cod SQL. Tabelul este în siguranță.

### `Parameters.AddWithValue` vs. `Parameters.Add`

`AddWithValue` este forma concisă — inferă tipul parametrului din valoarea .NET furnizată:

```csharp
cmd.Parameters.AddWithValue("@Titlu", carte.Titlu);       // string → NVARCHAR
```

`Parameters.Add` cu specificarea explicită a tipului SQL este mai precis și preferat în cod de producție, dar pentru exercițiu `AddWithValue` este suficient:

```csharp
// Varianta mai explicita
cmd.Parameters.Add("@Titlu", SqlDbType.NVarChar, 200).Value = carte.Titlu;
```

### `INSERT` complet

```csharp
public void Add(Carte c)
{
    using (SqlConnection conn = new SqlConnection(CONNECTION_STRING))
    {
        conn.Open();

        using (SqlCommand cmd = new SqlCommand(
            @"INSERT INTO Carti (Id, Titlu, Autor, AnAparitie, Gen)
              VALUES (@Id, @Titlu, @Autor, @AnAparitie, @Gen)", conn))
        {
            cmd.Parameters.AddWithValue("@Id", c.Id);
            cmd.Parameters.AddWithValue("@Titlu", c.Titlu);
            cmd.Parameters.AddWithValue("@Autor", c.Autor);
            cmd.Parameters.AddWithValue("@AnAparitie", c.AnAparitie);
            cmd.Parameters.AddWithValue("@Gen", c.Gen.ToString()); // enum → string
           
            cmd.ExecuteNonQuery();
        }
    }
}
```

`Gen` este stocat ca `NVARCHAR` în baza de date — la inserare îl convertim în string cu `.ToString()`. La citire vom face operația inversă.

### `UPDATE` complet

```csharp
public void Update(Carte c)
{
    using (SqlConnection conn = new SqlConnection(CONNECTION_STRING))
    {
        conn.Open();

        // Actualizam toate coloanele editabile, identificam randul dupa Id
        using (SqlCommand cmd = new SqlCommand(
            @"UPDATE Carti
              SET Titlu = @Titlu,
                  Autor = @Autor,
                  AnAparitie = @AnAparitie,
                  Gen = @Gen
              WHERE Id = @Id", conn))
        {
            cmd.Parameters.AddWithValue("@Id", c.Id);
            cmd.Parameters.AddWithValue("@Titlu", c.Titlu);
            cmd.Parameters.AddWithValue("@Autor", c.Autor);
            cmd.Parameters.AddWithValue("@AnAparitie", c.AnAparitie);
            cmd.Parameters.AddWithValue("@Gen", c.Gen.ToString());

            cmd.ExecuteNonQuery();
        }
    }
}
```

Clauza `WHERE Id = @Id` asigură că modificăm exact rândul dorit. Fără `WHERE`, am actualiza toate rândurile din tabel.

### `DELETE` complet

```csharp
public void Delete(int id)
{
    using (SqlConnection conn = new SqlConnection(CONNECTION_STRING))
    {
        conn.Open();

        using (SqlCommand cmd = new SqlCommand(
            "DELETE FROM Carti WHERE Id = @Id", conn))
        {
            cmd.Parameters.AddWithValue("@Id", id);
            cmd.ExecuteNonQuery();
        }
    }
}
```


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-7-ado.net/sqlcommand-si-executenonquery.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
