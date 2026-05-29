# Formulare multiple și comunicarea între ele

O aplicație reală rareori încape într-o singură fereastră. Operațiile de introducere a datelor, setările, rapoartele, dialogurile de confirmare — toate au sens pe ferestre separate, care se deschid la nevoie și se închid după ce și-au îndeplinit rolul.

În exercițiu, `Form1` este fereastra principală care afișează lista angajaților. `FormAngajat` este o fereastră secundară care apare când utilizatorul vrea să adauge sau să editeze un angajat. Cele două ferestre trebuie să comunice: `Form1` trimite date către `FormAngajat` (angajatul de editat) și primește date înapoi (`AngajatRezultat`).

### `Show()` vs. `ShowDialog()` — modal și non-modal

Există două moduri de a deschide un formular secundar.

**`Show()` — non-modal:**

```csharp
FormAngajat f = new FormAngajat(null);
f.Show();
// codul continua imediat aici, fara sa astepte inchiderea lui f
```

Formularul se deschide și codul din `Form1` continuă să ruleze imediat. Utilizatorul poate interacționa cu ambele ferestre simultan. Este potrivit pentru ferestre care rămân deschise în paralel, cum ar fi un panou de instrumente sau un log în timp real.

**`ShowDialog()` — modal:**

```csharp
DialogResult rezultat = f.ShowDialog();
// codul se opreste AICI pana cand utilizatorul inchide f
// dupa inchidere, rezultat contine ce a ales utilizatorul
```

Formularul se deschide și blochează complet fereastra care l-a deschis. Utilizatorul nu poate interacționa cu `Form1` cât timp `FormAngajat` este deschis. Este varianta corectă pentru dialoguri de adăugare, editare și confirmare.

> 📸 **SCREENSHOT:** Aplicația cu `FormAngajat` deschis deasupra `Form1`. Arată că `Form1` este vizibil în fundal dar nu poate fi accesat (titlul lui apare estompat sau bara de titlu e inactivă). Evidențiază că `FormAngajat` este în prim-plan și activ.

### Transmiterea datelor către formularul secundar

Cel mai clar mecanism este prin **constructor**. `FormAngajat` primește datele de care are nevoie la creare:

```csharp
// Adaugare: transmitem null — formularul stie ca e gol
using (FormAngajat f = new FormAngajat(null))
{
    if (f.ShowDialog() == DialogResult.OK)
    {
        // ...
    }
}

// Editare: transmitem angajatul de editat
Angajat selectat = angajati[dgvAngajati.SelectedRows[0].Index];
using (FormAngajat f = new FormAngajat(selectat))
{
    if (f.ShowDialog() == DialogResult.OK)
    {
        // ...
    }
}
```

`FormAngajat` primește parametrul în constructor și decide ce face cu el:

```csharp
public partial class FormAngajat : Form
{
    private Angajat angajatDeEditat;

    public FormAngajat(Angajat angajat)
    {
        InitializeComponent();
        angajatDeEditat = angajat;
    }

    private void FormAngajat_Load(object sender, EventArgs e)
    {
        if (angajatDeEditat != null)
        {
            // Modul editare: completam campurile cu datele existente
            this.Text = "Editează angajat";
            txtNume.Text = angajatDeEditat.Nume;
            txtPrenume.Text = angajatDeEditat.Prenume;
            // ...
        }
        else
        {
            // Modul adaugare: campuri goale
            this.Text = "Angajat nou";
        }
    }
}
```

Alternativa la constructor este prin **proprietăți publice**: setezi proprietățile după `new FormAngajat()` și înainte de `ShowDialog()`. Constructorul este mai clar și mai sigur — datele sunt disponibile de la momentul creării obiectului.

### Citirea rezultatului din formularul secundar

Există două mecanisme complementare.

**`DialogResult`** — răspunsul la întrebarea „utilizatorul a confirmat sau a anulat?":

```csharp
using (FormAngajat f = new FormAngajat(null))
{
    DialogResult rezultat = f.ShowDialog();

    if (rezultat == DialogResult.OK)
    {
        // utilizatorul a apasat OK si datele sunt valide
        Angajat nou = f.AngajatRezultat;
        angajati.Add(nou);
    }
    // daca rezultat == DialogResult.Cancel, nu facem nimic
}
```

**Proprietatea publică `AngajatRezultat`** — datele introduse de utilizator:

```csharp
// In FormAngajat:
public Angajat AngajatRezultat { get; private set; }

private void btnOk_Click(object sender, EventArgs e)
{
    if (!Valideaza()) return;  // nu inchidem daca validarea esueaza

    AngajatRezultat = new Angajat
    {
        Nume = txtNume.Text.Trim(),
        Prenume = txtPrenume.Text.Trim(),
        // ...
    };

    this.DialogResult = DialogResult.OK;  // inchide formularul
}
```

Combinarea celor două: `DialogResult.OK` garantează că `AngajatRezultat` a fost populat. Codul din `Form1` citește proprietatea doar după ce verifică `DialogResult`.

### `using` cu formulare

Formularul secundar implementează `IDisposable` și alocă resurse de sistem (handles Windows). Folosirea `using` garantează eliberarea lor:

```csharp
// Fara using — resursa nu e eliberata imediat
FormAngajat f = new FormAngajat(null);
f.ShowDialog();

// Cu using — eliberat automat la iesirea din bloc
using (FormAngajat f = new FormAngajat(null))
{
    if (f.ShowDialog() == DialogResult.OK)
    {
        Angajat nou = f.AngajatRezultat;
        angajati.Add(nou);
    }
}  // f.Dispose() este apelat automat aici
```

`using` este patternul recomandat pentru orice formular deschis cu `ShowDialog()`.

### Accesul la fereastra părinte

Uneori formularul secundar are nevoie de referință la cel care l-a deschis. `ShowDialog(this)` transmite referința la fereastra curentă ca `Owner`:

```csharp
f.ShowDialog(this);  // this = Form1, devine Owner al lui FormAngajat
```

Din `FormAngajat`, fereastra care l-a deschis este accesibilă prin `this.Owner`:

```csharp
Form1 parinte = this.Owner as Form1;
```

În exercițiu nu avem nevoie de acest mecanism, dar este util de știut pentru situații mai complexe.


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-5-datagridview-aplicatii-cu-ferestre-multiple/formulare-multiple-si-comunicarea-intre-ele.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
