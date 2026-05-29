# DataGridView - afișare și configurare de bază

`DataGridView` este controlul principal pentru afișarea datelor tabulare în WinForms. Afișează o colecție de obiecte ca un tabel cu rânduri și coloane, similar cu un Excel simplificat. Este mult mai potrivit decât `ListBox` atunci când ai mai multe proprietăți de afișat simultan.

### `DataSource` și `BindingList<T>`

Cel mai simplu și mai practic mod de a popula un `DataGridView` este prin proprietatea `DataSource`. Când atribui o colecție ca `DataSource`, grila generează automat câte o coloană pentru fiecare proprietate publică a tipului.

`BindingList<T>` este tipul preferat față de `List<T>` pentru că notifică automat `DataGridView`-ul când se adaugă sau se șterg elemente — grila se actualizează fără să apelezi manual `Refresh()`:

```csharp
public partial class Form1 : Form
{
    // BindingList in loc de List — actualizari automate in DataGridView
    private BindingList<Angajat> angajati = new BindingList<Angajat>();

    private void Form1_Load(object sender, EventArgs e)
    {
        // Atribuim DataSource o singura data, in Load
        dgvAngajati.DataSource = angajati;

        // Adaugam cateva date de test
        angajati.Add(new Angajat { Nume = "Popescu", Prenume = "Ion", Departament = "IT", Salariu = 5000 });
        angajati.Add(new Angajat { Nume = "Ionescu", Prenume = "Maria", Departament = "HR", Salariu = 4200 });
        // Grila se actualizeaza automat
    }
}
```

> 📸 **SCREENSHOT:** `DataGridView` populat cu câteva rânduri de angajați, arătând că fiecare proprietate a clasei `Angajat` a generat automat o coloană. Evidențiați antetele coloanelor.

### Coloane auto-generate vs. configurate manual

Când setezi `DataSource`, `DataGridView`-ul generează o coloană pentru fiecare proprietate publică, inclusiv cele pe care nu vrei să le afișezi. Poți controla asta în două moduri.

**Varianta simplă — ascunderea coloanelor după generare:**

```csharp
private void Form1_Load(object sender, EventArgs e)
{
    dgvAngajati.DataSource = angajati;

    // Ascundem coloanele pe care nu vrem sa le afisam
    dgvAngajati.Columns["EstePermanent"].Visible = false;

    // Redenumim antetele
    dgvAngajati.Columns["Nume"].HeaderText = "Nume";
    dgvAngajati.Columns["DataAngajarii"].HeaderText = "Data angajării";
    dgvAngajati.Columns["Salariu"].HeaderText = "Salariu (RON)";
}
```

**Varianta controlată — dezactivezi auto-generarea și adaugi coloane manual:**

```csharp
dgvAngajati.AutoGenerateColumns = false;

// Adaugam doar coloanele dorite
dgvAngajati.Columns.Add(new DataGridViewTextBoxColumn
{
    DataPropertyName = "Prenume",  // proprietatea din Angajat
    HeaderText = "Prenume",
    Width = 120
});
dgvAngajati.Columns.Add(new DataGridViewTextBoxColumn
{
    DataPropertyName = "Departament",
    HeaderText = "Departament",
    Width = 100
});
```

Pentru exercițiu, varianta simplă cu ascunderea coloanelor nedorite este suficientă.

### Proprietăți esențiale de configurare

```csharp
dgvAngajati.ReadOnly = true;
// Celulele nu pot fi editate direct in grid — editarea se face prin FormAngajat

dgvAngajati.SelectionMode = DataGridViewSelectionMode.FullRowSelect;
// Selectia unui rand selecteaza intreaga linie, nu doar o celula

dgvAngajati.MultiSelect = false;
// Permite selectarea unui singur rand la un moment dat

dgvAngajati.AutoSizeColumnsMode = DataGridViewAutoSizeColumnsMode.Fill;
// Coloanele se intind sa umple latimea disponibila a controlului

dgvAngajati.AllowUserToAddRows = false;
// Ascunde randul gol de la final (randul de adaugare rapida)

dgvAngajati.AllowUserToDeleteRows = false;
// Impiedica stergerea randurilor cu tasta Delete direct in grid

dgvAngajati.RowHeadersVisible = false;
// Ascunde coloana gri cu sagetile din stanga
```

> 📸 **SCREENSHOT:** Același `DataGridView` cu și fără `RowHeadersVisible = false` și `AllowUserToAddRows = false`, arătând cum arată grila cu și fără rândul de adăugare și fără antetul de rânduri.

### Obținerea elementului selectat

Când `DataSource` este o `BindingList<T>`, indexul rândului selectat corespunde direct indexului în colecție:

```csharp
if (dgvAngajati.SelectedRows.Count == 0)
{
    MessageBox.Show("Selecteaza un angajat.");
    return;
}

// Indexul randului selectat in grid = indexul in BindingList
int index = dgvAngajati.SelectedRows[0].Index;
Angajat selectat = angajati[index];
```

Alternativ, poți folosi `CurrentRow`:

```csharp
if (dgvAngajati.CurrentRow == null) return;
int index = dgvAngajati.CurrentRow.Index;
```

> 📸 **SCREENSHOT:** `DataGridView` cu un rând selectat (evidențiat în albastru pe toată lățimea). Arată că `SelectionMode = FullRowSelect` selectează întreaga linie, nu doar o celulă.

### Evenimentul `SelectionChanged`

Util pentru a activa sau dezactiva butoanele de editare și ștergere în funcție de dacă ceva este selectat:

```csharp
private void dgvAngajati_SelectionChanged(object sender, EventArgs e)
{
    bool esteSelectat = dgvAngajati.SelectedRows.Count > 0;
    btnEditeaza.Enabled = esteSelectat;
    btnSterge.Enabled = esteSelectat;
}
```

Apelează această metodă și în `Form1_Load` pentru a seta starea inițială corectă (butoanele dezactivate când lista e goală).

### Evenimentul `CellDoubleClick`

O îmbunătățire de UX: dublu-click pe un rând deschide direct formularul de editare:

```csharp
private void dgvAngajati_CellDoubleClick(object sender, DataGridViewCellEventArgs e)
{
    // e.RowIndex -1 inseamna click pe antet — ignoram
    if (e.RowIndex < 0) return;

    btnEditeaza_Click(sender, EventArgs.Empty);  // reutilizam logica din buton
}
```


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-5-datagridview-aplicatii-cu-ferestre-multiple/datagridview-afisare-si-configurare-de-baza.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
