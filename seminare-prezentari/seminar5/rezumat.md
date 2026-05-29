# Rezumat

### 1. Formulare multiple și comunicarea între ele

| Concept                              | Ce înseamnă                                                           |
| ------------------------------------ | --------------------------------------------------------------------- |
| **`Show()`**                         | Deschide non-modal — ambele ferestre active simultan                  |
| **`ShowDialog()`**                   | Deschide modal — blochează fereastra părinte până la închidere        |
| **Constructor cu parametru**         | Mecanismul preferat de transmitere a datelor spre formularul secundar |
| **Proprietate publică**              | Mecanismul de returnare a datelor din formularul secundar             |
| **`using (var f = new Form()) { }`** | Garantează `Dispose()` după `ShowDialog()`                            |

Pattern complet de deschidere:

```csharp
using (FormAngajat f = new FormAngajat(angajatSauNull))
{
    if (f.ShowDialog() == DialogResult.OK)
    {
        // f.AngajatRezultat este disponibil aici
    }
}
```

### 2. `DialogResult` și ciclul de viață

| Concept                                  | Ce înseamnă                                                        |
| ---------------------------------------- | ------------------------------------------------------------------ |
| **`this.DialogResult = OK`**             | Închide formularul și returnează `OK` apelantului                  |
| **`AcceptButton`**                       | Butonul activat la `Enter`                                         |
| **`CancelButton`**                       | Butonul activat la `Escape` și la X                                |
| **`btnOk.DialogResult = None`**          | Necesar când ai validare în handler                                |
| **Obiect accesibil după `ShowDialog()`** | Citiți `AngajatRezultat` în blocul `using`, înainte de `Dispose()` |

Ciclul de viață: `new` → `Load` → interacțiune → `DialogResult = OK/Cancel` → `ShowDialog()` returnează → `Dispose()`.

### 3. Reutilizarea formularului pentru adăugare și editare

| Concept                                     | Ce înseamnă                                                     |
| ------------------------------------------- | --------------------------------------------------------------- |
| **Constructor cu `null`**                   | Modul adăugare — câmpuri goale                                  |
| **Constructor cu obiect**                   | Modul editare — câmpuri precompletate în `Form_Load`            |
| **`AngajatRezultat { get; private set; }`** | Proprietate publică citită de `Form1` după `ShowDialog()`       |
| **Obiect nou în `btnOk_Click`**             | Modificările aplicate doar dacă `DialogResult == OK`            |
| **`angajati[index] = f.AngajatRezultat`**   | Înlocuire la editare — `BindingList` actualizează grila automat |

```csharp
// Adaugare
using (FormAngajat f = new FormAngajat(null))
    if (f.ShowDialog() == DialogResult.OK)
        angajati.Add(f.AngajatRezultat);

// Editare
using (FormAngajat f = new FormAngajat(angajati[index]))
    if (f.ShowDialog() == DialogResult.OK)
        angajati[index] = f.AngajatRezultat;
```

### 4. `DataGridView`

| Proprietate                     | Ce face                                              |
| ------------------------------- | ---------------------------------------------------- |
| `DataSource = bindingList`      | Leagă grila de colecție — actualizări automate       |
| `ReadOnly = true`               | Previne editarea directă în celule                   |
| `SelectionMode = FullRowSelect` | Selecția unui rând selectează toată linia            |
| `MultiSelect = false`           | Permite un singur rând selectat                      |
| `AutoSizeColumnsMode = Fill`    | Coloanele umplu lățimea disponibilă                  |
| `AllowUserToAddRows = false`    | Ascunde rândul gol de adăugare rapidă                |
| `SelectedRows[0].Index`         | Indexul rândului selectat = indexul în `BindingList` |

| Eveniment          | Când se declanșează                                             |
| ------------------ | --------------------------------------------------------------- |
| `SelectionChanged` | La orice schimbare a selecției — activează/dezactivează butoane |
| `CellDoubleClick`  | La dublu-click — deschide editarea direct                       |

### 5. `ComboBox` și `DateTimePicker`

| Concept                    | `ComboBox`                                        | `DateTimePicker`                                 |
| -------------------------- | ------------------------------------------------- | ------------------------------------------------ |
| **Proprietate principală** | `SelectedItem`, `SelectedIndex`                   | `Value`                                          |
| **Setare inițială**        | `SelectedIndex = 0`                               | `Value = DateTime.Today`                         |
| **Citire**                 | `SelectedItem.ToString()`                         | `Value.Date`                                     |
| **Eveniment principal**    | `SelectedIndexChanged`                            | `ValueChanged`                                   |
| **Stil recomandat**        | `DropDownStyle = DropDownList` pentru valori fixe | `Format = Custom`, `CustomFormat = "dd.MM.yyyy"` |

### 6. Validare cu `ErrorProvider`

| Concept                                  | Ce înseamnă                                                   |
| ---------------------------------------- | ------------------------------------------------------------- |
| **Validarea în `FormAngajat`**           | Nu în `Form1` — formularul secundar cunoaște regulile proprii |
| **`errorProvider1.SetError(ctrl, msg)`** | Afișează iconița de eroare lângă control                      |
| **`errorProvider1.Clear()`**             | Șterge toate erorile înainte de revalidare                    |
| **Validare fără oprire la prima eroare** | Marchează toate câmpurile cu probleme simultan                |
| **`decimal.TryParse`**                   | Conversie sigură din text la număr, fără excepție             |

```csharp
private bool Valideaza()
{
    errorProvider1.Clear();
    bool esteValid = true;

    if (string.IsNullOrWhiteSpace(txtNume.Text))
    { errorProvider1.SetError(txtNume, "Obligatoriu."); esteValid = false; }

    decimal salariu;
    if (!decimal.TryParse(txtSalariu.Text.Trim(), out salariu) || salariu <= 0)
    { errorProvider1.SetError(txtSalariu, "Numar pozitiv."); esteValid = false; }

    return esteValid;
}
```


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-5-datagridview-aplicatii-cu-ferestre-multiple/rezumat.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
