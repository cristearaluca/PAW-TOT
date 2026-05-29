# Exercițiu

Construiești un catalog de cărți cu o arhitectură separată în straturi. Datele trăiesc într-o clasă statică `FakeDatabase`, care simulează o sursă de date externă. Repository-ul este un intermediar care expune operațiile CRUD către restul aplicației. Formularele de UI nu accesează niciodată `FakeDatabase` direct.

### Modelul, baza de date și repository-ul

**Enumerația `GenCarte`:**

* Valorile: `Roman`, `Stiinta`, `Fictiune`, `Biografie`, `Tehnic`, `Altele`

**Clasa `Carte`:**

* `Guid Id`
* `string Titlu`
* `string Autor`
* `int AnAparitie`
* `GenCarte Gen`

**Clasa statică `FakeDatabase`:**

* O singură proprietate statică: `static List<Carte> Carti`, inițializată cu câteva cărți de test
* Nu conține nicio logică, este doar un container de date

**Clasa `CarteRepository`:**

Implementează metodele CRUD care accesează exclusiv `FakeDatabase.Carti`:

* `List<Carte> GetAll()` - returnează toate cărțile din listă
* `Carte GetById(Guid id)` - returnează cartea cu id-ul dat sau `null`
* `void Add(Carte c)` - adaugă în `FakeDatabase.Carti`
* `void Update(Carte c)` - înlocuiește cartea cu același `Id`
* `void Delete(Guid id)` - elimină cartea cu id-ul dat

Repository-ul nu conține validare și nu știe nimic despre UI.

### Fereastra principală cu `ListView`

Pe `Form1` adaugă:

* Un `ListView` (`lvCarti`) configurat în modul `Details` cu coloanele: `Titlu`, `Autor`, `An`, `Gen` cu lățimile 30%, 30%, 20% și 20% (folosește proprietatea `lvCarti.ClientSize.Width`)
* Trei butoane: `btnAdauga`, `btnEditeaza`, `btnSterge`
* Un `Label` de status (`lblStatus`)

Configurează `lvCarti`:

* `View = Details`
* `FullRowSelect = true`
* `MultiSelect = false`
* `GridLines = true`
* `AllowColumnReorder = true`

Declară în `Form1` un `CarteRepository` privat și instanțiează-l în constructor.

Implementează metoda `RefreshLista()` care golește `lvCarti` și repopulează din `repository.GetAll()`. Stochează referința la obiectul `Carte` în proprietatea `Tag` a fiecărui `ListViewItem`.

Adaugă coloanele și apelează `RefreshLista()` în `Form1_Load`. Dezactivează `btnEditeaza` și `btnSterge` la pornire.

### Formularul secundar `FormCarte`

Pe `FormCarte` adaugă:

* `TextBox` pentru Titlu (`txtTitlu`)
* `TextBox` pentru Autor (`txtAutor`)
* `NumericUpDown` pentru An apariție (`numAn`) - minim 1000, maxim anul curent
* `ComboBox` pentru Gen (`cmbGen`) - populat din enum cu `Enum.GetValues`
* Butoane `btnOk` și `btnAnuleaza` (legate la `AcceptButton` și `CancelButton` ale `FormCarte`)
* Un `ErrorProvider` (`epCarte`)

`FormCarte` primește în constructor doar `Guid? id` - `null` pentru adăugare, id-ul cărții pentru editare. Instanțiază propriul `CarteRepository` intern.

În `Form_Load`:

* Dacă `id` este `null`: mod adăugare, câmpuri goale, `Text = "Carte nouă"`
* Dacă `id` are valoare: mod editare, completează câmpurile din `repository.GetById(id.Value)`, `Text = "Editează carte"`

La click pe `btnOk`:

* Validează câmpurile cu `ErrorProvider` (Titlu și Autor obligatorii, An și Preț numere valide)
* Dacă adăugare: creează un `Carte` și apelează `repository.Add()`
* Dacă editare: preia obiectul cu `GetById`, modifică proprietățile, apelează `repository.Update()`
* Setează `this.DialogResult = DialogResult.OK`

### Operațiile CRUD în `Form1`

**Adăugare:**

```csharp
using (var f = new FormCarte())
    if (f.ShowDialog() == DialogResult.OK)
        RefreshLista();
```

**Editare:**

* Verifică dacă un element este selectat
* Obține cartea selectată din `lvCarti.SelectedItems[0].Tag`
* Deschide `FormCarte` transmițând `id`-ul cărții
* Dacă `DialogResult.OK`, apelează `RefreshLista()`

**Ștergere:**

* Verifică selecția
* Afișează `MessageBox` de confirmare cu titlul cărții
* Dacă confirmat, apelează `repository.Delete(id)` și `RefreshLista()`

Activează și dezactivează `btnEditeaza` și `btnSterge` în `SelectedIndexChanged`.


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-6-separarea-responsabilitatilor-listview/exercitiu.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
