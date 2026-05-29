# ComboBox

`ComboBox` combină un câmp text cu o listă derulantă. Utilizatorul poate fie alege o valoare din listă, fie (opțional) introduce una nouă. Este controlul potrivit când ai un set finit de valori cunoscute — departamente, categorii, statusuri.

#### Popularea cu valori

**Varianta simplă — valori fixe adăugate în cod:**

```csharp
cmbDepartament.Items.Add("IT");
cmbDepartament.Items.Add("HR");
cmbDepartament.Items.Add("Financiar");
cmbDepartament.Items.Add("Vânzări");

// Sau cu AddRange pentru mai multe valori dintr-o data
cmbDepartament.Items.AddRange(new string[] { "IT", "HR", "Financiar", "Vânzări" });
```

**Varianta cu `DataSource`** — pentru liste din colecții sau enumerații:

```csharp
// Din array
cmbDepartament.DataSource = new string[] { "IT", "HR", "Financiar", "Vânzări" };

// Din enum
cmbDepartament.DataSource = Enum.GetValues(typeof(Departament));
```

Popularea se face în `Form_Load`, nu în constructor, pentru că controalele sunt disponibile și formularul este gata de afișat.

> 📸 **SCREENSHOT:** `ComboBox` deschis (lista derulantă vizibilă) cu valorile departamentelor listate. Arată cum arată atât starea închisă cât și cea deschisă.

#### `DropDownStyle`

Controlează dacă utilizatorul poate introduce valori libere sau doar alege din listă:

```csharp
cmbDepartament.DropDownStyle = ComboBoxStyle.DropDownList;
// Utilizatorul poate DOAR alege din lista — nu poate tasta liber
// Recomandat cand setul de valori este fix si controlat

cmbDepartament.DropDownStyle = ComboBoxStyle.DropDown;
// Utilizatorul poate alege SAU tasta o valoare noua (comportament implicit)
```

Pentru departamente, `DropDownList` este alegerea corectă — nu vrem valori libere introduse de utilizator.

#### Selectarea și citirea valorii

```csharp
// Selectare prin index (primul element, index 0)
cmbDepartament.SelectedIndex = 0;

// Selectare prin valoare
cmbDepartament.SelectedItem = "IT";

// Citire — valoarea selectata ca object, necesita ToString()
string departament = cmbDepartament.SelectedItem.ToString();

// Verificare ca ceva e selectat
if (cmbDepartament.SelectedItem == null)
{
    MessageBox.Show("Selecteaza un departament.");
    return;
}
```

La precompletarea câmpurilor în modul editare, atribuim direct valoarea string:

```csharp
cmbDepartament.SelectedItem = angajatDeEditat.Departament;
// Daca valoarea exista in Items, e selectata automat
// Daca nu exista, SelectedItem ramane null
```

#### Evenimentul `SelectedIndexChanged`

Se declanșează când selecția se schimbă. Util pentru filtrare sau pentru a actualiza alte controale în funcție de selecție:

```csharp
private void cmbFiltruDepartament_SelectedIndexChanged(object sender, EventArgs e)
{
    string departamentSelectat = cmbFiltruDepartament.SelectedItem.ToString();

    if (departamentSelectat == "Toate")
    {
        dgvAngajati.DataSource = angajati;
    }
    else
    {
        var filtrati = angajati.Where(a => a.Departament == departamentSelectat).ToList();
        dgvAngajati.DataSource = new BindingList<Angajat>(filtrati);
    }
}
```

> 📸 **SCREENSHOT:** Fereastra principală cu `cmbFiltruDepartament` selectat pe „IT" și `DataGridView`-ul afișând doar angajații din departamentul IT. Arată că filtrarea funcționează în timp real.


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-5-datagridview-aplicatii-cu-ferestre-multiple/combobox.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
