# ErrorProvider

Validarea datelor introduse de utilizator se face în `FormAngajat`, nu în `Form1`. Motivul este simplu: `FormAngajat` știe ce câmpuri are și ce reguli trebuie respectate. `Form1` nu ar trebui să aibă acces direct la câmpurile din `FormAngajat` și nici să cunoască regulile de validare.

Structura corectă:

```
btnOk_Click (FormAngajat)
  -> Valideaza()
      -> daca esueaza: marcheaza campul, afiseaza eroarea, return false
      -> daca trece: return true
  -> daca Valideaza() returneaza false: return (formularul ramane deschis)
  -> construieste AngajatRezultat
  -> this.DialogResult = DialogResult.OK (formularul se inchide)
```

### Validare simplă cu `MessageBox`

Varianta de bază: verifici câmpurile în ordine și afișezi un `MessageBox` la prima eroare găsită:

```csharp
private bool Valideaza()
{
    if (string.IsNullOrWhiteSpace(txtNume.Text))
    {
        MessageBox.Show("Numele este obligatoriu.", "Eroare",
            MessageBoxButtons.OK, MessageBoxIcon.Warning);
        txtNume.Focus();
        return false;
    }

    if (string.IsNullOrWhiteSpace(txtPrenume.Text))
    {
        MessageBox.Show("Prenumele este obligatoriu.", "Eroare",
            MessageBoxButtons.OK, MessageBoxIcon.Warning);
        txtPrenume.Focus();
        return false;
    }

    if (cmbDepartament.SelectedItem == null)
    {
        MessageBox.Show("Selecteaza un departament.", "Eroare",
            MessageBoxButtons.OK, MessageBoxIcon.Warning);
        cmbDepartament.Focus();
        return false;
    }

    decimal salariu;
    if (!decimal.TryParse(txtSalariu.Text.Trim(), out salariu) || salariu <= 0)
    {
        MessageBox.Show("Salariul trebuie sa fie un numar pozitiv.", "Eroare",
            MessageBoxButtons.OK, MessageBoxIcon.Warning);
        txtSalariu.Focus();
        txtSalariu.SelectAll();
        return false;
    }

    return true;
}
```

Această variantă funcționează, dar afișează o fereastră de dialog separată pentru fiecare eroare. Utilizatorul trebuie să dea OK, să corecteze, să dea OK din nou etc.

***

### `ErrorProvider` — validare inline

`ErrorProvider` este un component care afișează o iconița roșie de eroare direct lângă controlul cu problema, fără ferestre de dialog suplimentare. Experiența utilizatorului este mai fluentă: vede toate erorile simultan și le poate corecta fără întreruperi.

#### Adăugarea în Designer

`ErrorProvider` se adaugă din Toolbox — nu este un control vizibil, ci un component care apare în bara de componente sub formular.

> 📸 **SCREENSHOT:** Designer-ul `FormAngajat` cu `ErrorProvider` adăugat. Arată bara de componente de sub formular cu `errorProvider1` vizibil. Evidențiați că `ErrorProvider` nu apare pe suprafața formularului, ci în zona de componente.

#### Utilizarea în cod

```csharp
// Seteaza o eroare pe un control — apare iconita rosie
errorProvider1.SetError(txtNume, "Numele este obligatoriu.");

// Sterge eroarea de pe un control
errorProvider1.SetError(txtNume, "");  // string gol = fara eroare

// Sterge toate erorile
errorProvider1.Clear();
```

#### Validare completă cu `ErrorProvider`

Spre deosebire de validarea cu `MessageBox`, nu mai oprești la prima eroare — verifici toate câmpurile și marchezi toate problemele simultan:

```csharp
private bool Valideaza()
{
    // Stergem toate erorile precedente inainte de o noua validare
    errorProvider1.Clear();

    bool esteValid = true;

    if (string.IsNullOrWhiteSpace(txtNume.Text))
    {
        errorProvider1.SetError(txtNume, "Numele este obligatoriu.");
        esteValid = false;
    }

    if (string.IsNullOrWhiteSpace(txtPrenume.Text))
    {
        errorProvider1.SetError(txtPrenume, "Prenumele este obligatoriu.");
        esteValid = false;
    }

    if (cmbDepartament.SelectedItem == null)
    {
        errorProvider1.SetError(cmbDepartament, "Selecteaza un departament.");
        esteValid = false;
    }

    decimal salariu;
    if (!decimal.TryParse(txtSalariu.Text.Trim(), out salariu) || salariu <= 0)
    {
        errorProvider1.SetError(txtSalariu, "Salariul trebuie sa fie un numar pozitiv.");
        esteValid = false;
    }

    // Daca validarea esueaza, mutam focus-ul pe primul camp cu eroare
    if (!esteValid)
        txtNume.Focus();

    return esteValid;
}
```

> 📸 **SCREENSHOT:** `FormAngajat` cu `ErrorProvider` activ: câmpul Nume gol cu iconița roșie de eroare vizibilă în dreapta lui, și tooltip-ul de eroare afișat la hover. Dacă poți, arată mai multe câmpuri cu erori simultan.

***

### Ștergerea erorilor la modificare

O îmbunătățire de UX: ștergi eroarea de pe un câmp imediat ce utilizatorul începe să corecteze:

```csharp
private void txtNume_TextChanged(object sender, EventArgs e)
{
    if (!string.IsNullOrWhiteSpace(txtNume.Text))
        errorProvider1.SetError(txtNume, "");
}
```

Sau mai concis, cu `sender`:

```csharp
// Acelasi handler abonat la mai multe TextBox-uri
private void txt_TextChanged(object sender, EventArgs e)
{
    TextBox txt = sender as TextBox;
    if (txt != null && !string.IsNullOrWhiteSpace(txt.Text))
        errorProvider1.SetError(txt, "");
}
```

***

### Validarea numerelor cu `TryParse`

Câmpurile pentru numere (Salariu) sunt `TextBox`-uri care conțin text. Conversia se face cu `TryParse` — mai sigur decât `Parse` deoarece nu aruncă excepție:

```csharp
// Parse — arunca exceptie daca textul nu e numar
decimal salariu = decimal.Parse(txtSalariu.Text);  // NU folosi asta in UI

// TryParse — returneaza false daca nu e numar, fara exceptie
decimal salariu;
if (!decimal.TryParse(txtSalariu.Text.Trim(), out salariu))
{
    errorProvider1.SetError(txtSalariu, "Introduceti un numar valid.");
    return false;
}

if (salariu <= 0)
{
    errorProvider1.SetError(txtSalariu, "Salariul trebuie sa fie pozitiv.");
    return false;
}
```


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-5-datagridview-aplicatii-cu-ferestre-multiple/errorprovider.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
