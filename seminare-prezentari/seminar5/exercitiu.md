# Exercițiu

Construiești o aplicație desktop pentru gestionarea unei liste de angajați. Fereastra principală afișează toți angajații într-un `DataGridView`. Adăugarea și editarea se fac printr-un formular secundar reutilizabil — același formular se deschide gol pentru adăugare și precompletat pentru editare. Datele sunt stocate în memorie pe durata sesiunii.

### Clasa `Angajat` și structura proiectului

Adaugă în proiect clasa `Angajat` cu proprietățile:

* `string Nume`
* `string Prenume`
* `string Departament`
* `decimal Salariu`
* `DateTime DataAngajarii`
* `bool EstePermanent`

Adaugă metoda `GetNumeComplet()` care returnează `"Prenume Nume"` și suprascrie `ToString()` cu același rezultat.

Proiectul va conține două formulare:

* `Form1` — fereastra principală cu lista de angajați
* `FormAngajat` — formularul secundar pentru adăugare și editare

Adaugă `FormAngajat` prin **Project → Add → Windows Form**.

### Fereastra principală cu `DataGridView`

Pe `Form1` adaugă:

* Un `DataGridView` (`dgvAngajati`) care ocupă cea mai mare parte a ferestrei
* Un `Label` + `ComboBox` (`cmbFiltruDepartament`) pentru filtrare, deasupra grilei
* Trei `Button`-uri sub grilă: `btnAdauga`, `btnEditeaza`, `btnSterge`
* Un `Label` de status (`lblStatus`) în partea de jos

Configurează `dgvAngajati`:

* `ReadOnly = true`
* `SelectionMode = FullRowSelect`
* `MultiSelect = false`
* `AutoSizeColumnsMode = Fill`

Declară în `Form1` o `BindingList<Angajat>` și asign-o ca `DataSource` al `dgvAngajati` în evenimentul `Load`.

Populează `cmbFiltruDepartament` cu valorile: `"Toate"`, `"IT"`, `"HR"`, `"Financiar"`, `"Vânzări"`. Selectează implicit `"Toate"`.

### Formularul secundar `FormAngajat`&#x20;

Pe `FormAngajat` adaugă:

* `Label` + `TextBox` pentru Nume (`txtNume`)
* `Label` + `TextBox` pentru Prenume (`txtPrenume`)
* `Label` + `ComboBox` pentru Departament (`cmbDepartament`) cu valorile: `"IT"`, `"HR"`, `"Financiar"`, `"Vânzări"`
* `Label` + `TextBox` pentru Salariu (`txtSalariu`)
* `Label` + `DateTimePicker` pentru Data angajării (`dtpDataAngajarii`)
* `CheckBox` pentru `"Angajat permanent"` (`chkEstePermanent`)
* `Button` OK (`btnOk`) și `Button` Anulează (`btnAnuleaza`)

Configurează formularul:

* `FormBorderStyle = FixedDialog`
* `StartPosition = CenterParent`
* `MaximizeBox = false`, `MinimizeBox = false`
* Setează `AcceptButton = btnOk` și `CancelButton = btnAnuleaza`

Adaugă în `FormAngajat`:

* O proprietate publică `Angajat AngajatRezultat` care va conține datele introduse
* Un constructor care primește un `Angajat` opțional (`null` pentru adăugare)
* Dacă `Angajat` primit nu este `null`, completează câmpurile cu datele lui și schimbă `Text`-ul formularului în `"Editează angajat"`
* Dacă este `null`, setează `Text` = `"Angajat nou"`

### Adăugarea unui angajat&#x20;

La click pe `btnAdauga` din `Form1`:

* Deschide `FormAngajat` cu `ShowDialog()`, transmițând `null`
* Dacă `DialogResult == DialogResult.OK`, adaugă `AngajatRezultat` în `BindingList`
* Actualizează `lblStatus`

În `FormAngajat`, la click pe `btnOk`:

* Validează că Nume, Prenume nu sunt goale și că Salariul este un număr pozitiv
* Dacă validarea eșuează, folosește `ErrorProvider` pentru a marca câmpul și nu închide formularul
* Dacă validarea reușește, construiește `AngajatRezultat` din valorile câmpurilor și setează `this.DialogResult = DialogResult.OK`

### Editarea unui angajat

La click pe `btnEditeaza` din `Form1`:

* Verifică dacă un rând este selectat; dacă nu, afișează `MessageBox`
* Obține `Angajat`-ul selectat din `BindingList` folosind `dgvAngajati.SelectedRows[0].Index`
* Deschide `FormAngajat` cu `ShowDialog()`, transmițând obiectul selectat
* Dacă `DialogResult == DialogResult.OK`, înlocuiește angajatul din `BindingList` cu `AngajatRezultat`

### Ștergere și filtrare

**Ștergere:** La click pe `btnSterge`:

* Verifică dacă un rând este selectat
* Afișează `MessageBox` cu confirmare `YesNo` care include numele angajatului
* Dacă confirmat, elimină angajatul din `BindingList`

**Filtrare după departament:** La `SelectedIndexChanged` pe `cmbFiltruDepartament`:

* Dacă valoarea selectată este `"Toate"`, setează `DataSource`-ul cu întreaga `BindingList`
* Altfel, construiește o listă filtrată și setează-o ca `DataSource` temporar


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-5-datagridview-aplicatii-cu-ferestre-multiple/exercitiu.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
