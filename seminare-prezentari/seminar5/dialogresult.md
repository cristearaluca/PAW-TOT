# DialogResult

`DialogResult` este o enumerație care reprezintă răspunsul unui formular modal la întrebarea implicită pe care o pune utilizatorului: „confirmi sau anulezi?". Fiecare buton de pe formular poate fi configurat să seteze un anumit `DialogResult` la apăsare, ceea ce închide automat formularul și returnează valoarea respectivă apelantului.

```csharp
// In Form1, dupa ShowDialog():
using (FormAngajat f = new FormAngajat(null))
{
    DialogResult rezultat = f.ShowDialog();

    if (rezultat == DialogResult.OK)
        angajati.Add(f.AngajatRezultat);
    // daca Cancel, nu facem nimic
}
```

### Valorile `DialogResult`

```csharp
DialogResult.OK       // utilizatorul a confirmat
DialogResult.Cancel   // utilizatorul a anulat sau a inchis cu X
DialogResult.Yes      // pentru YesNo
DialogResult.No       // pentru YesNo
DialogResult.Retry    // pentru RetryCancel
DialogResult.Abort    // pentru AbortRetryIgnore
DialogResult.Ignore   // pentru AbortRetryIgnore
DialogResult.None     // formularul nu a fost inca inchis
```

În aplicații cu formulare de adăugare/editare, `OK` și `Cancel` sunt suficiente în aproape toate cazurile.

### Setarea `DialogResult` din cod

Atribuirea unui `DialogResult` pe formular îl închide automat:

```csharp
private void btnOk_Click(object sender, EventArgs e)
{
    if (!Valideaza()) return;  // validarea esueaza — formularul ramane deschis

    AngajatRezultat = CitesteFormular();
    this.DialogResult = DialogResult.OK;  // inchide formularul
    // nu mai e nevoie de this.Close() — setarea DialogResult il inchide automat
}

private void btnAnuleaza_Click(object sender, EventArgs e)
{
    this.DialogResult = DialogResult.Cancel;  // inchide formularul
}
```

Dacă utilizatorul închide fereastra cu butonul X din bara de titlu, `DialogResult` devine automat `Cancel`. Acest comportament implicit este exact ce vrem.

### `AcceptButton` și `CancelButton`

Două proprietăți ale `Form` simplifică interacțiunea cu tastatura:

**`AcceptButton`** — butonul activat când utilizatorul apasă `Enter`, indiferent de controlul cu focus:

```csharp
this.AcceptButton = btnOk;
```

**`CancelButton`** — butonul activat când utilizatorul apasă `Escape`:

```csharp
this.CancelButton = btnAnuleaza;
```

Se setează din fereastra Properties a formularului sau din cod după `InitializeComponent()`. Cu aceste două setate, utilizatorul poate confirma cu `Enter` și anula cu `Escape` fără să dea click pe butoane.

> 📸 **SCREENSHOT:** Fereastra Properties a `FormAngajat` cu proprietățile `AcceptButton` și `CancelButton` evidențiate și valorile `btnOk` și `btnAnuleaza` selectate din dropdown.

Un detaliu important: dacă `btnOk` are `DialogResult = OK` setat din Designer (în Properties → `DialogResult`), apăsarea lui închide formularul **fără** să treacă prin `btnOk_Click`. Acesta este comportamentul nedorit când ai validare — validarea nu ar mai rula.

Soluția: lasă `btnOk.DialogResult = None` în Designer și setează `this.DialogResult = DialogResult.OK` manual în handler, **după** validare:

```csharp
// GRESIT — DialogResult setat in Designer pe buton
// Formularul se inchide la click, validarea nu ruleaza

// CORECT — DialogResult setat din cod, dupa validare
private void btnOk_Click(object sender, EventArgs e)
{
    if (!Valideaza()) return;
    AngajatRezultat = CitesteFormular();
    this.DialogResult = DialogResult.OK;  // validarea a trecut, acum inchidem
}
```

> 📸 **SCREENSHOT:** Fereastra Properties a `btnOk` cu proprietatea `DialogResult` evidențiată și valoarea setată la `None`. Adaugă o notă că aceasta este setarea corectă când ai validare în handler.

### Ciclul de viață al unui formular secundar

Un formular deschis cu `ShowDialog()` parcurge acești pași:

```
new FormAngajat(angajat)
  -> constructor
  -> InitializeComponent()
  -> [FormAngajat_Load]  — daca e abonat
  -> fereastra apare pe ecran
  -> utilizatorul interactioneaza
  -> [btnOk_Click sau btnAnuleaza_Click]
  -> this.DialogResult = OK / Cancel
  -> fereastra dispare de pe ecran
  -> ShowDialog() returneaza DialogResult
  -> [using] Dispose() elibereaza resursele
```

Un aspect important: după ce `ShowDialog()` returnează, obiectul formularului nu a dispărut — el există în memorie până la `Dispose()`. Poți accesa proprietăți publice ale lui, inclusiv `AngajatRezultat`, cât timp ești în blocul `using`:

```csharp
using (FormAngajat f = new FormAngajat(null))
{
    if (f.ShowDialog() == DialogResult.OK)
    {
        // f inca exista in memorie, AngajatRezultat este accesibil
        Angajat nou = f.AngajatRezultat;
        angajati.Add(nou);
    }
}
// aici f.Dispose() a fost apelat, f nu mai e accesibil
```

### `FormClosing` în formularul secundar

Dacă vrei să interzici închiderea cu X fără confirmare, te abonezi la `FormClosing`:

```csharp
private void FormAngajat_FormClosing(object sender, FormClosingEventArgs e)
{
    // Daca utilizatorul a apasat OK, lasam sa se inchida
    if (this.DialogResult == DialogResult.OK) return;

    // Daca a apasat X sau Escape si are date introduse, cerem confirmare
    if (!string.IsNullOrWhiteSpace(txtNume.Text))
    {
        DialogResult confirmare = MessageBox.Show(
            "Ai date nesalvate. Inchizi fara sa salvezi?",
            "Confirmare",
            MessageBoxButtons.YesNo,
            MessageBoxIcon.Question);

        if (confirmare == DialogResult.No)
            e.Cancel = true;  // anuleaza inchiderea
    }
}
```


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-5-datagridview-aplicatii-cu-ferestre-multiple/dialogresult.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
