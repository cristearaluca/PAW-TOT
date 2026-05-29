# Ce este ADO.NET?

ADO.NET este biblioteca din .NET Framework pentru accesul la baze de date relaționale. Oferă un set de clase care permit trimiterea de comenzi SQL către o bază de date și primirea rezultatelor înapoi, direct din cod C#.

Față de alternativele mai moderne precum Entity Framework — care generează automat cod SQL din clase C# — ADO.NET este explicit și manual: tu scrii interogările SQL, tu deschizi și închizi conexiunea, tu citești rezultatele coloană cu coloană. Aceasta înseamnă mai mult cod, dar și control total și transparență completă asupra a ce se întâmplă în baza de date.

Clasele principale din ADO.NET pentru SQL Server se află în namespace-ul `System.Data.SqlClient` și trebuie adăugat la începutul fișierului:

```csharp
using System.Data.SqlClient;
```

### Ierarhia claselor principale

Orice operație cu baza de date în ADO.NET implică aceleași trei clase, în aceeași ordine:

**`SqlConnection`** — reprezintă conexiunea fizică la baza de date. Fără o conexiune deschisă, nu poți trimite nicio comandă. Conexiunea consumă resurse de sistem și trebuie închisă imediat ce nu mai este necesară.

**`SqlCommand`** — reprezintă comanda SQL pe care vrei să o execuți. Poate fi un `SELECT`, `INSERT`, `UPDATE` sau `DELETE`. Comanda este asociată unei conexiuni și poate conține parametri.

**`SqlDataReader`** — reprezintă rezultatul unui `SELECT`. Funcționează ca un cursor care parcurge rândurile returnate unul câte unul. Este disponibil doar atâta timp cât conexiunea este deschisă.

Relația dintre ele:

```
SqlConnection → SqlCommand → SqlDataReader
     ↑                ↑              ↑
  "Usa de             "Ce            "Rezultatele
   intrare"           intrebi"       citite rand
                                     cu rand"
```

### Connection string

**Connection string**-ul este un șir de text care descrie cum să te conectezi la baza de date: unde se află, cu ce credențiale, cu ce opțiuni. Este echivalentul unei adrese plus a unei chei de acces.

Pentru o bază de date SQL Server în fișier `.mdf` folosind LocalDB, connection string-ul arată astfel:

```csharp
private const string CONNECTION_STRING =
    @"Data Source=(LocalDB)\MSSQLLocalDB;" +
    @"AttachDbFilename=|DataDirectory|\CartiDB.mdf;" +
    @"Integrated Security=True";
```

Fiecare parte are un rol precis:

`Data Source=(LocalDB)\MSSQLLocalDB` — specifică instanța SQL Server la care ne conectăm. LocalDB este o versiune ușoară de SQL Server instalată odată cu Visual Studio, destinată dezvoltării locale. `MSSQLLocalDB` este numele instanței implicite.

`AttachDbFilename=|DataDirectory|\CartiDB.mdf` — specifică fișierul de bază de date. `|DataDirectory|` este un placeholder special care este înlocuit automat cu calea directorului de date al aplicației — de obicei folderul `bin\Debug` unde rulează executabilul. Aceasta face connection string-ul portabil: nu depinde de o cale absolută.

`Integrated Security=True` — înseamnă că autentificarea se face cu credențialele Windows ale utilizatorului curent, nu cu un user și parolă SQL Server separate.

### Setarea `DataDirectory` în `Program.cs`

Pentru ca `|DataDirectory|` să fie rezolvat corect la runtime, trebuie să îi spui aplicației care este calea. Aceasta se face în `Program.cs`, înainte de lansarea formularului principal:

```csharp
static class Program
{
    [STAThread]
    static void Main()
    {
        // Setam DataDirectory la folderul unde ruleaza executabilul
        // Aceasta face ca |DataDirectory| din connection string sa fie rezolvat corect
        AppDomain.CurrentDomain.SetData("DataDirectory", Application.StartupPath);

        Application.EnableVisualStyles();
        Application.SetCompatibleTextRenderingDefault(false);
        Application.Run(new Form1());
    }
}
```

Fără această linie, `|DataDirectory|` nu este rezolvat și conexiunea eșuează cu o eroare de tipul „cannot attach the file".

### Deschiderea și închiderea conexiunii

O conexiune la baza de date consumă resurse atât în aplicație cât și pe serverul de baze de date. Regula fundamentală este: **deschide conexiunea cât mai târziu posibil și închide-o cât mai devreme posibil**.

Cel mai sigur mod de a gestiona o conexiune este cu `using`. Blocul `using` garantează că `Dispose()` este apelat la ieșire, ceea ce include și închiderea conexiunii — chiar dacă apare o excepție în mijlocul blocului:

```csharp
// Fara using — conexiunea nu e inchisa daca apare o exceptie
SqlConnection conn = new SqlConnection(CONNECTION_STRING);
conn.Open();
// ... daca apare exceptie aici, conn.Close() nu mai e apelat niciodata
conn.Close();

// Cu using — conexiunea e inchisa garantat, indiferent ce se intampla
using (SqlConnection conn = new SqlConnection(CONNECTION_STRING))
{
    conn.Open();
    // ... orice exceptie apare, conn.Dispose() e apelat automat la iesirea din bloc
}
```

Aceeași regulă se aplică și pentru `SqlCommand` și `SqlDataReader`.

### Structura de bază a unei operații ADO.NET

Indiferent că faci un `SELECT`, `INSERT`, `UPDATE` sau `DELETE`, structura este aceeași:

```csharp
using (SqlConnection conn = new SqlConnection(CONNECTION_STRING))
{
    conn.Open();  // deschide conexiunea

    using (SqlCommand cmd = new SqlCommand("...", conn))
    {
        // configurezi comanda si adaugi parametri
        // executi comanda
    }
    // conexiunea se inchide automat la iesirea din using
}
```

Concret, pentru un `DELETE`:

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

Fiecare metodă din repository deschide propria conexiune, o folosește și o închide. Nu există o conexiune persistentă la nivel de câmp în repository — aceasta ar crea probleme de resurse și de concurență.


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-7-ado.net/ce-este-ado.net.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
