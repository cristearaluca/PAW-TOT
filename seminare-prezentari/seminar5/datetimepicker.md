# DateTimePicker

`DateTimePicker` este un control specializat pentru selectarea datelor calendaristice. Afișează o dată și permite modificarea ei printr-un calendar pop-up sau prin tastare directă.

#### Proprietăți esențiale

```csharp
// Citirea datei selectate
DateTime dataAngajarii = dtpDataAngajarii.Value;

// Setarea unei valori initiale
dtpDataAngajarii.Value = DateTime.Today;
dtpDataAngajarii.Value = angajatDeEditat.DataAngajarii;  // la editare

// Limitarea intervalului acceptat
dtpDataAngajarii.MinDate = new DateTime(2000, 1, 1);
dtpDataAngajarii.MaxDate = DateTime.Today;  // nu permite date viitoare
```

#### Formatarea afișată

`DateTimePicker` poate afișa data în mai multe formate:

```csharp
// Format scurt: 12.03.2024
dtpDataAngajarii.Format = DateTimePickerFormat.Short;

// Format lung: luni, 12 martie 2024
dtpDataAngajarii.Format = DateTimePickerFormat.Long;

// Format personalizat
dtpDataAngajarii.Format = DateTimePickerFormat.Custom;
dtpDataAngajarii.CustomFormat = "dd.MM.yyyy";
```

> 📸 **SCREENSHOT:** `DateTimePicker` în două stări: (1) afișând data în format scurt, (2) cu calendarul pop-up deschis după click pe săgeata dropdown. Arată că utilizatorul poate naviga prin luni și selecta o zi.

#### `DateTimePicker` cu opțiunea „fără dată"

Proprietatea `ShowCheckBox` adaugă o casetă de bifat care permite selectarea „fără dată" (`null`):

```csharp
dtpDataAngajarii.ShowCheckBox = true;
dtpDataAngajarii.Checked = false;  // "fara data" selectat

// Citire cu verificare
if (dtpDataAngajarii.Checked)
    DateTime data = dtpDataAngajarii.Value;
else
    // data e goala — trateaza ca null
```

#### Utilizarea în `FormAngajat`

La inițializare:

```csharp
private void FormAngajat_Load(object sender, EventArgs e)
{
    // Format vizual
    dtpDataAngajarii.Format = DateTimePickerFormat.Custom;
    dtpDataAngajarii.CustomFormat = "dd.MM.yyyy";

    if (angajatDeEditat != null)
        dtpDataAngajarii.Value = angajatDeEditat.DataAngajarii;
    else
        dtpDataAngajarii.Value = DateTime.Today;
}
```

La citire:

```csharp
AngajatRezultat = new Angajat
{
    // ...
    DataAngajarii = dtpDataAngajarii.Value.Date  // .Date elimina componenta de timp
};
```


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-5-datagridview-aplicatii-cu-ferestre-multiple/datetimepicker.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
