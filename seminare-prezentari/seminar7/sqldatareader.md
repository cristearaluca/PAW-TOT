# SqlDataReader

`SqlDataReader` este mecanismul de citire a rândurilor returnate de o interogare `SELECT`. Spre deosebire de `ExecuteNonQuery()` care nu returnează rânduri, sau `ExecuteScalar()` care returnează o singură valoare, `ExecuteReader()` returnează un `SqlDataReader` cu care parcurgi rândurile rezultatului unul câte unul.

`SqlDataReader` este un **cursor forward-only** — poți parcurge rândurile o singură dată, de la primul la ultimul, fără să te poți întoarce. Aceasta îl face eficient în memorie: nu încarcă toate rezultatele dintr-o dată, ci le citește pe rând din baza de date.

### Structura de bază cu `while (reader.Read())`

```csharp
using (SqlConnection conn = new SqlConnection(CONNECTION_STRING))
{
    conn.Open();

    using (SqlCommand cmd = new SqlCommand(
        "SELECT Id, Titlu, Autor, AnAparitie, Gen FROM Carti", conn))
    {
        using (SqlDataReader reader = cmd.ExecuteReader())
        {
            // reader.Read() avanseaza cursorul la urmatorul rand
            // returneaza true daca mai exista randuri, false daca s-au terminat
            while (reader.Read())
            {
                // Aici procesezi randul curent
                // reader["NumeColoana"] returneaza valoarea coloanei ca object
            }
        }
    }
}
```

`reader.Read()` face două lucruri simultan: avansează cursorul la următorul rând și returnează `true` dacă rândul există, `false` dacă nu mai sunt rânduri. Bucla `while` se oprește automat după ultimul rând.

### Citirea valorilor dintr-un rând

Odată ce `reader.Read()` a poziționat cursorul pe un rând, ai acces la valorile coloanelor prin mai multe mecanisme:

**Prin numele coloanei — cel mai lizibil:**

```csharp
object valoare = reader["Titlu"];  // returneaza object
string titlu = reader["Titlu"].ToString();
Guid id = (Guid)reader["Id"];
```

**Prin metodele tipizate — mai sigur și mai eficient:**

```csharp
Guid id       = reader.GetGuid(reader.GetOrdinal("Id"));
string titlu = reader.GetString(reader.GetOrdinal("Titlu"));
```

`GetOrdinal("NumeColoana")` returnează indexul coloanei după nume. `GetInt32(index)`, `GetString(index)`, `GetDecimal(index)` citesc valoarea direct în tipul corect, fără boxing/unboxing.

Pentru exercițiu, citirea prin numele coloanei cu cast explicit este mai simplă și perfect funcțională:

```csharp
while (reader.Read())
{
    Carte c = new Carte();
    c.Id          = (Guid)reader["Id"];
    c.Titlu       = reader["Titlu"].ToString();
    c.Autor       = reader["Autor"].ToString();
    c.AnAparitie  = (int)reader["AnAparitie"];
    c.Gen         = (GenCarte)Enum.Parse(typeof(GenCarte), reader["Gen"].ToString());

    carti.Add(c);
}
```

### Conversia enum-ului din string

Coloana `Gen` este stocată ca `NVARCHAR` în baza de date — un string. La citire, trebuie să îl convertiți înapoi la tipul `GenCarte`. Metoda `Enum.Parse` face această conversie:

```csharp
string genString = reader["Gen"].ToString();  // "Tehnic", "Roman" etc.
GenCarte gen = (GenCarte)Enum.Parse(typeof(GenCarte), genString);
```

`Enum.Parse` aruncă excepție dacă string-ul nu corespunde niciunei valori din enum. Dacă există risc că datele din baza de date au valori neașteptate, poți folosi `Enum.TryParse` cu o valoare implicită:

```csharp
GenCarte gen;
if (!Enum.TryParse(reader["Gen"].ToString(), out gen))
    gen = GenCarte.Altele;  // valoare implicita daca string-ul nu e recunoscut
```

### `GetAll()`&#x20;

```csharp
public List<Carte> GetAll()
{
    List<Carte> carti = new List<Carte>();

    using (SqlConnection conn = new SqlConnection(CONNECTION_STRING))
    {
        conn.Open();

        using (SqlCommand cmd = new SqlCommand(
            "SELECT Id, Titlu, Autor, AnAparitie, Gen FROM Carti", conn))
        {
            using (SqlDataReader reader = cmd.ExecuteReader())
            {
                while (reader.Read())
                {
                    Carte c = new Carte();
                    c.Id         = (Guid)reader["Id"];
                    c.Titlu      = reader["Titlu"].ToString();
                    c.Autor      = reader["Autor"].ToString();
                    c.AnAparitie = (int)reader["AnAparitie"];
                    c.Gen        = (GenCarte)Enum.Parse(
                                       typeof(GenCarte), reader["Gen"].ToString());
            
                    carti.Add(c);
                }
            }
        }
    }

    return carti;
}
```

### `GetById()`&#x20;

Pentru a citi un singur rând, adaugi o clauză `WHERE` și verifici dacă `reader.Read()` returnează `true` la primul apel:

```csharp
public Carte GetById(Guid id)
{
    using (SqlConnection conn = new SqlConnection(CONNECTION_STRING))
    {
        conn.Open();

        using (SqlCommand cmd = new SqlCommand(
            "SELECT Id, Titlu, Autor, AnAparitie, Gen FROM Carti WHERE Id = @Id", conn))
        {
            cmd.Parameters.AddWithValue("@Id", id);

            using (SqlDataReader reader = cmd.ExecuteReader())
            {
                // Apelam Read() o singura data — exista cel mult un rand cu Id-ul dat
                if (reader.Read())
                {
                    Carte c = new Carte();
                    c.Id         = (Guid)reader["Id"];
                    c.Titlu      = reader["Titlu"].ToString();
                    c.Autor      = reader["Autor"].ToString();
                    c.AnAparitie = (int)reader["AnAparitie"];
                    c.Gen        = (GenCarte)Enum.Parse(
                                       typeof(GenCarte), reader["Gen"].ToString());
                    return c;
                }

                return null;  // niciun rand nu a indeplinit conditia WHERE
            }
        }
    }
}
```

Diferența față de `GetAll()` este că folosim `if (reader.Read())` în loc de `while (reader.Read())` — ne așteptăm la cel mult un rând, nu la o colecție.

### De ce reader-ul trebuie închis înainte de a reutiliza conexiunea

Un `SqlDataReader` deschis menține o referință la conexiunea activă. Atâta timp cât reader-ul este deschis, nu poți executa o altă comandă pe aceeași conexiune. Blocul `using` garantează că reader-ul este închis la ieșire:

```csharp
using (SqlDataReader reader = cmd.ExecuteReader())
{
    while (reader.Read()) { ... }
}
// reader este inchis automat aici — conexiunea poate fi reutilizata
```

Dacă ar trebui să execuți o a doua comandă în timp ce reader-ul este deschis, fie deschizi o a doua conexiune, fie încarci toate datele din reader într-o colecție și îl închizi înainte de a executa a doua comandă.


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-7-ado.net/sqldatareader.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
