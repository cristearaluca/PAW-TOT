# Repository cu ADO.NET

În laboratorul anterior, `CarteRepository` accesa `FakeDatabase.Carti` — o listă statică în memorie. Interfața publică a repository-ului era identică: `GetAll`, `GetById`, `Add`, `Update`, `Delete`. Formularele de UI nu știau că datele vin dintr-o listă.

Acum rescriim implementarea metodelor să acceseze o bază de date SQL Server. **Interfața rămâne identică** — aceleași metode, aceleași semnături. Formularele de UI nu se modifică, pentru că ele comunică cu repository-ul prin aceeași interfață publică pe care au folosit-o întotdeauna.

Aceasta este demonstrația concretă a valorii separării straturilor: înlocuim complet sursa de date fără să atingem niciun formular.

### Structura completă a repository-ului

```csharp
using System;
using System.Collections.Generic;
using System.Data.SqlClient;

class CarteRepository
{
    // Connection string-ul ca o constanta privata a clasei.
    // |DataDirectory| este inlocuit cu Application.StartupPath setat in Program.cs.
    private const string CONNECTION_STRING =
        @"Data Source=(LocalDB)\MSSQLLocalDB;" +
        @"AttachDbFilename=|DataDirectory|\CartiDB.mdf;" +
        @"Integrated Security=True";

    // Metoda privata helper pentru a construi un obiect Carte dintr-un reader.
    // Evita duplicarea aceluiasi cod de citire in GetAll si GetById.
    private Carte CitesteCarte(SqlDataReader reader)
    {
        Carte c = new Carte();
        c.Id         = (Guid)reader["Id"];
        c.Titlu      = reader["Titlu"].ToString();
        c.Autor      = reader["Autor"].ToString();
        c.AnAparitie = (int)reader["AnAparitie"];
        c.Gen        = (GenCarte)Enum.Parse(typeof(GenCarte), reader["Gen"].ToString());
        return c;
    }

    public List<Carte> GetAll()
    {
        List<Carte> carti = new List<Carte>();
        try
        {
            using (SqlConnection conn = new SqlConnection(CONNECTION_STRING))
            {
                conn.Open();
                using (SqlCommand cmd = new SqlCommand(
                    "SELECT Id, Titlu, Autor, AnAparitie, Gen FROM Carti", conn))
                using (SqlDataReader reader = cmd.ExecuteReader())
                {
                    while (reader.Read())
                        carti.Add(CitesteCarte(reader));
                }
            }
        }
        catch (SqlException ex)
        {
            throw new Exception("Eroare la incarcarea cartilor: " + ex.Message, ex);
        }
        return carti;
    }

    public Carte GetById(Guid id)
    {
        try
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
                        if (reader.Read())
                            return CitesteCarte(reader);
                        return null;
                    }
                }
            }
        }
        catch (SqlException ex)
        {
            throw new Exception("Eroare la cautarea cartii: " + ex.Message, ex);
        }
    }

    public void Add(Carte c)
    {
        try
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
                    cmd.Parameters.AddWithValue("@Gen", c.Gen.ToString());

                    cmd.ExecuteNonQuery();
                }
            }
        }
        catch (SqlException ex)
        {
            throw new Exception("Eroare la adaugarea cartii: " + ex.Message, ex);
        }
    }

    public void Update(Carte c)
    {
        try
        {
            using (SqlConnection conn = new SqlConnection(CONNECTION_STRING))
            {
                conn.Open();
                using (SqlCommand cmd = new SqlCommand(
                    @"UPDATE Carti
                      SET Titlu      = @Titlu,
                          Autor      = @Autor,
                          AnAparitie = @AnAparitie,
                          Gen        = @Gen,
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
        catch (SqlException ex)
        {
            throw new Exception("Eroare la actualizarea cartii: " + ex.Message, ex);
        }
    }

    public void Delete(Guid id)
    {
        try
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
        catch (SqlException ex)
        {
            throw new Exception("Eroare la stergerea cartii: " + ex.Message, ex);
        }
    }
}
```

### Metoda helper `CitesteCarte`

Observă că `GetAll()` și `GetById()` construiesc un obiect `Carte` din reader în mod identic. Dacă duplicăm codul de citire în ambele metode, orice modificare — de exemplu, adăugarea unei noi coloane — trebuie făcută în două locuri.

Metoda privată `CitesteCarte(SqlDataReader reader)` extrage această logică o singură dată. Ambele metode de citire o apelează. Aceasta este aceeași regulă de la prezentare — nu duplica codul, extrage în metode.

### Gestionarea excepțiilor în repository

Erorile de baze de date vin ca `SqlException` — un tip specializat pentru erori SQL Server. Repository-ul prinde `SqlException` și aruncă mai departe o `Exception` generică cu un mesaj mai clar.

De ce rethrowing în loc de gestionare directă? Repository-ul nu știe cum să afișeze erori — nu are `MessageBox`, nu are label de status. Știe doar că ceva a mers prost. Stratul de prezentare este cel care decide cum comunică eroarea utilizatorului.

```csharp
// Repository — prinde eroarea SQL, adauga context, o arunca mai departe
catch (SqlException ex)
{
    throw new Exception("Eroare la adaugarea cartii: " + ex.Message, ex);
}

// Form1 — prinde exceptia si o afiseaza utilizatorului
private void btnAdauga_Click(object sender, EventArgs e)
{
    try
    {
        using (var f = new FormCarte(null))
        {
            if (f.ShowDialog() == DialogResult.OK)
            {
                RefreshLista();
                lblStatus.Text = "Carte adaugata.";
            }
        }
    }
    catch (Exception ex)
    {
        MessageBox.Show(ex.Message, "Eroare", MessageBoxButtons.OK, MessageBoxIcon.Error);
    }
}
```

Parametrul `ex` din `throw new Exception(..., ex)` este **inner exception** — păstrează excepția originală în lanț. Dacă cineva vrea să vadă detaliile tehnice ale erorii SQL, le poate accesa prin `ex.InnerException`.

### Setarea `DataDirectory` în `Program.cs`&#x20;

```csharp
static class Program
{
    [STAThread]
    static void Main()
    {
        // Fara aceasta linie, |DataDirectory| din connection string nu e rezolvat
        // si aplicatia nu gaseste fisierul .mdf
        AppDomain.CurrentDomain.SetData("DataDirectory", Application.StartupPath);

        Application.EnableVisualStyles();
        Application.SetCompatibleTextRenderingDefault(false);
        Application.Run(new Form1());
    }
}
```


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-7-ado.net/repository-cu-ado.net.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
