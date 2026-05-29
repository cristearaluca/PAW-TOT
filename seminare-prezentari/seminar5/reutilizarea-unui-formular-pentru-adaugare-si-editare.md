# Reutilizarea unui formular pentru adăugare și editare

Adăugarea și editarea unui angajat sunt aproape identice ca interfață: același set de câmpuri, aceeași validare, aceleași butoane. Diferența este că la editare câmpurile vin precompletate și la confirmare se actualizează un obiect existent, nu se creează unul nou.

A crea două formulare separate (`FormAdaugaAngajat` și `FormEditeazaAngajat`) înseamnă să duplicăm layout-ul, controalele și validarea. Orice modificare ulterioară — un câmp nou, o regulă de validare schimbată — trebuie aplicată în două locuri.

Soluția este un singur `FormAngajat` care știe în ce mod funcționează în funcție de ce primește în constructor.

### Constructorul cu parametru opțional

`FormAngajat` primește un `Angajat` prin constructor. Convenția: `null` înseamnă adăugare, un obiect înseamnă editare.

```csharp
public partial class FormAngajat : Form
{
    // Stocam referinta la angajatul de editat (null daca adaugam)
    private Angajat angajatDeEditat;

    // Proprietatea publica prin care Form1 citeste rezultatul
    public Angajat AngajatRezultat { get; private set; }

    public FormAngajat(Angajat angajat)
    {
        InitializeComponent();
        angajatDeEditat = angajat;
    }
}
```

### Completarea câmpurilor în `Form_Load`

Logica de inițializare a câmpurilor se pune în evenimentul `Load`, nu în constructor. La momentul constructorului, controalele există (au fost create de `InitializeComponent()`), dar formularul nu este încă afișat. `Load` este momentul potrivit pentru setup vizual.

```csharp
private void FormAngajat_Load(object sender, EventArgs e)
{
    // Populam ComboBox-ul cu departamentele — acelasi indiferent de mod
    cmbDepartament.Items.AddRange(new string[] { "IT", "HR", "Financiar", "Vânzări" });

    if (angajatDeEditat != null)
    {
        // MOD EDITARE: completam campurile cu datele existente
        this.Text = "Editează angajat";
        txtNume.Text = angajatDeEditat.Nume;
        txtPrenume.Text = angajatDeEditat.Prenume;
        cmbDepartament.SelectedItem = angajatDeEditat.Departament;
        txtSalariu.Text = angajatDeEditat.Salariu.ToString("F2");
        dtpDataAngajarii.Value = angajatDeEditat.DataAngajarii;
        chkEstePermanent.Checked = angajatDeEditat.EstePermanent;
    }
    else
    {
        // MOD ADAUGARE: campuri goale cu valori implicite
        this.Text = "Angajat nou";
        cmbDepartament.SelectedIndex = 0;
        dtpDataAngajarii.Value = DateTime.Today;
    }
}
```

> 📸 **SCREENSHOT:** Două instanțe ale `FormAngajat` alăturate: una în modul „Angajat nou" cu câmpuri goale, una în modul „Editează angajat" cu câmpuri precompletate. Arată că titlul ferestrei diferă și că datele din modul editare corespund unui angajat din grid.

### Construirea `AngajatRezultat` la confirmare

La click pe `btnOk`, dacă validarea trece, construim obiectul rezultat și setăm `DialogResult`:

```csharp
private void btnOk_Click(object sender, EventArgs e)
{
    if (!Valideaza()) return;

    // Construim angajatul cu datele din campuri
    AngajatRezultat = new Angajat
    {
        Nume = txtNume.Text.Trim(),
        Prenume = txtPrenume.Text.Trim(),
        Departament = cmbDepartament.SelectedItem.ToString(),
        Salariu = decimal.Parse(txtSalariu.Text.Trim()),
        DataAngajarii = dtpDataAngajarii.Value,
        EstePermanent = chkEstePermanent.Checked
    };

    this.DialogResult = DialogResult.OK;
}
```

Creăm întotdeauna un obiect nou, indiferent de modul de funcționare. Nu modificăm `angajatDeEditat` direct — `Form1` decide ce face cu `AngajatRezultat`.

### Utilizarea în `Form1` — adăugare vs. editare

Diferența dintre adăugare și editare este în `Form1`, nu în `FormAngajat`:

```csharp
// ADAUGARE
private void btnAdauga_Click(object sender, EventArgs e)
{
    using (FormAngajat f = new FormAngajat(null))  // null = mod adaugare
    {
        if (f.ShowDialog() == DialogResult.OK)
        {
            angajati.Add(f.AngajatRezultat);  // adaugam in lista
            lblStatus.Text = "Angajat adaugat: " + f.AngajatRezultat.GetNumeComplet();
        }
    }
}

// EDITARE
private void btnEditeaza_Click(object sender, EventArgs e)
{
    if (dgvAngajati.SelectedRows.Count == 0)
    {
        MessageBox.Show("Selecteaza un angajat pentru editare.", "Atentie",
            MessageBoxButtons.OK, MessageBoxIcon.Information);
        return;
    }

    int index = dgvAngajati.SelectedRows[0].Index;
    Angajat selectat = angajati[index];

    using (FormAngajat f = new FormAngajat(selectat))  // selectat = mod editare
    {
        if (f.ShowDialog() == DialogResult.OK)
        {
            angajati[index] = f.AngajatRezultat;  // inlocuim in lista la acelasi index
            lblStatus.Text = "Angajat actualizat: " + f.AngajatRezultat.GetNumeComplet();
        }
    }
}
```

### De ce `AngajatRezultat` este un obiect nou

Ai putea fi tentat să modifici direct `angajatDeEditat` în `FormAngajat`:

```csharp
// Tentant dar problematic
angajatDeEditat.Nume = txtNume.Text;
angajatDeEditat.Prenume = txtPrenume.Text;
// ...
this.DialogResult = DialogResult.OK;
```

Problema: dacă utilizatorul apasă Cancel după ce a modificat câmpurile, `angajatDeEditat` a fost deja modificat în memorie, chiar dacă nu voia să salveze. Creând un obiect nou și returnându-l prin `AngajatRezultat`, `Form1` aplică modificările **numai** dacă `DialogResult == OK`.


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-5-datagridview-aplicatii-cu-ferestre-multiple/reutilizarea-unui-formular-pentru-adaugare-si-editare.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
