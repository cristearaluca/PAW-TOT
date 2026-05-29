# ListView in modul Details

`ListView` este un control care poate afișa o colecție de elemente în mai multe moduri vizuale, controlate prin proprietatea `View`: `LargeIcon`, `SmallIcon`, `List`, `Tile` și `Details`. Modul `Details` afișează datele ca un tabel cu rânduri și coloane — similar cu panoul din Windows Explorer în vizualizarea detalii.

Față de controalele pe care le-am văzut anterior, fiecare are locul lui:

* **`ListBox`** afișează o singură coloană de text, generată automat din `ToString()`. Potrivit când ai un singur atribut de afișat sau când `ToString()` rezumă bine obiectul și nu ai nevoie de coloane separate.
* **`DataGridView`** se leagă direct la o sursă de date prin `DataSource` și generează coloane automat din proprietățile tipului. Potrivit când vrei binding automat și actualizări fără cod suplimentar, și nu ai nevoie de control fin asupra formatului afișat.
* **`ListView` în modul `Details`** se populează manual, rând cu rând, din cod. Nu există binding automat — tu construiești fiecare rând explicit. Această abordare îți oferă control complet: alegi exact ce coloane apar, cum sunt formatate valorile și ce stocat în fiecare celulă. Este o alegere bună când prezentarea datelor contează — de exemplu, când vrei să afișezi prețul cu simbol monetar, data formatată sau un câmp calculat care nu există ca proprietate pe obiect.

### Configurarea de bază

Modul `Details` se setează prin proprietatea `View`, iar câteva alte proprietăți îl fac utilizabil pentru o aplicație de gestiune:

```csharp
lvCarti.View = View.Details;
// Modul tabel cu coloane — cel mai util pentru liste de obiecte cu mai multe atribute

lvCarti.FullRowSelect = true;
// Un click pe orice celula selecteaza intregul rand, nu doar celula respectiva.
// Fara aceasta, utilizatorul poate selecta o singura celula din mijlocul unui rand,
// ceea ce este confuz si face recuperarea obiectului la selectare mai complicata.

lvCarti.MultiSelect = false;
// Permite selectarea unui singur rand la un moment dat.
// Simplifica logica de editare si stergere — nu trebuie sa gestionezi selectii multiple.

lvCarti.GridLines = true;
// Afiseaza linii de separare intre randuri — mai usor de citit pentru liste lungi.

lvCarti.AllowColumnReorder = true;
// Optional: permite utilizatorului sa reordoneze coloanele prin drag and drop.
```

### Adăugarea coloanelor

Coloanele se adaugă în cod, de obicei în `Form1_Load`, înainte de popularea cu date. Metoda `Columns.Add` primește textul antetului, lățimea în pixeli și alinierea textului din coloană:

```csharp
private void Form1_Load(object sender, EventArgs e)
{
    // Adaugam coloanele — ordinea in care le adaugam este ordinea in care vor aparea
    lvCarti.Columns.Add("Titlu", 200, HorizontalAlignment.Left);
    lvCarti.Columns.Add("Autor", 150, HorizontalAlignment.Left);
    lvCarti.Columns.Add("An", 60, HorizontalAlignment.Center);
    lvCarti.Columns.Add("Gen", 100, HorizontalAlignment.Left);
    lvCarti.Columns.Add("Preț", 80, HorizontalAlignment.Right);

    // Populam cu date abia dupa ce coloanele sunt definite
    RefreshLista();
}
```

Alinierea textului din coloană este un detaliu vizual care comunică tipul datei: numerele și prețurile se aliniează natural la dreapta (exact cum scriem cifrele în tabele financiare), textele la stânga, codurile sau anii scurți se centrează.

### `ListViewItem` și sub-items

Un rând în `ListView` este reprezentat de un obiect `ListViewItem`. Prima coloană a rândului este textul principal al item-ului — transmis direct în constructor. Coloanele următoare se adaugă ca `SubItems`, în ordinea în care apar coloanele:

```csharp
// Construim un rand pentru o carte
ListViewItem item = new ListViewItem(carte.Titlu);         // coloana 1: Titlu
item.SubItems.Add(carte.Autor);                            // coloana 2: Autor
item.SubItems.Add(carte.AnAparitie.ToString());            // coloana 3: An
item.SubItems.Add(carte.Gen.ToString());                   // coloana 4: Gen
item.SubItems.Add(carte.Pret.ToString("F2") + " RON");    // coloana 5: Pret

// Adaugam randul in ListView
lvCarti.Items.Add(item);
```

Ordinea `SubItems.Add` trebuie să corespundă exact ordinii coloanelor definite anterior. Dacă adaugi coloanele în ordinea Titlu, Autor, An, Gen, Preț, atunci sub-items-urile trebuie adăugate în aceeași ordine — altfel datele vor apărea în coloanele greșite fără niciun mesaj de eroare.

### Proprietatea `Tag`&#x20;

Aceasta este tehnica esențială care face `ListView`-ul util în aplicații CRUD, și merită înțeleasă bine.

Când utilizatorul selectează un rând și apasă `Editează` sau `Șterge`, ai nevoie să știi exact ce obiect `Carte` corespunde rândului selectat. Ai putea căuta cartea în repository după titlu, dar aceasta este fragil — pot exista două cărți cu același titlu. Ai putea reține indexul rândului, dar indexul se schimbă dacă lista este filtrată sau sortată.

Soluția elegantă este proprietatea `Tag` de tip `object`, disponibilă pe orice `ListViewItem`. Poți stoca în ea orice referință — în cazul nostru, obiectul `Carte` corespunzător rândului:

```csharp
ListViewItem item = new ListViewItem(carte.Titlu);
item.SubItems.Add(carte.Autor);
// ... restul sub-items ...

item.Tag = carte;  // stocam referinta directa la obiectul Carte

lvCarti.Items.Add(item);
```

Când utilizatorul selectează rândul, recuperezi obiectul direct din `Tag`, fără căutări suplimentare:

```csharp
if (lvCarti.SelectedItems.Count == 0) return;

// Tag este de tip object — facem cast la Carte cu operatorul as
Carte selectata = lvCarti.SelectedItems[0].Tag as Carte;

// Acum avem acces direct la toate proprietatile obiectului
using (var f = new FormCarte(selectata.Id))
    f.ShowDialog();
```

`Tag` nu afectează nimic vizual — este un spațiu de stocare invizibil atașat rândului, exclusiv pentru uz intern în cod. Utilizatorul nu știe că există.

### Metoda `RefreshLista()`

Patternul de populare este același ca la `ListBox` din laboratorul anterior — o metodă dedicată care golește și repopulează de fiecare dată:

```csharp
private void RefreshLista()
{
    lvCarti.Items.Clear();

    foreach (Carte c in repository.GetAll())
    {
        ListViewItem item = new ListViewItem(c.Titlu);
        item.SubItems.Add(c.Autor);
        item.SubItems.Add(c.AnAparitie.ToString());
        item.SubItems.Add(c.Gen.ToString());
        item.SubItems.Add(c.Pret.ToString("F2") + " RON");
        item.Tag = c;  // esential — fara Tag nu putem recupera obiectul la selectare

        lvCarti.Items.Add(item);
    }

    lblStatus.Text = "Total: " + repository.GetAll().Count + " carti.";
}
```

Această metodă se apelează după orice operație care modifică datele: adăugare, editare, ștergere. Nu există binding automat ca la `DataGridView` cu `BindingList` — actualizarea este manuală, dar și explicită.

### Obținerea elementului selectat și activarea butoanelor

`SelectedItems` este o colecție de `ListViewItem`. Chiar dacă `MultiSelect = false`, verificăm `Count > 0` înainte să accesăm `[0]` — accesul la indexul 0 pe o colecție goală aruncă excepție.

```csharp
private void btnEditeaza_Click(object sender, EventArgs e)
{
    if (lvCarti.SelectedItems.Count == 0)
    {
        MessageBox.Show("Selecteaza o carte pentru editare.", "Atentie",
            MessageBoxButtons.OK, MessageBoxIcon.Information);
        return;
    }

    // Recuperam obiectul din Tag — cast cu as, returneaza null daca tipul nu se potriveste
    Carte selectata = lvCarti.SelectedItems[0].Tag as Carte;

    using (var f = new FormCarte(selectata.Id))
    {
        if (f.ShowDialog() == DialogResult.OK)
        {
            RefreshLista();
            lblStatus.Text = "Carte actualizata.";
        }
    }
}
```

O îmbunătățire importantă de UX: dezactivează butoanele de editare și ștergere când nimic nu este selectat și activează-le la selecție, prin evenimentul `SelectedIndexChanged`. Utilizatorul primește feedback vizual clar despre ce poate face în starea curentă:

```csharp
private void lvCarti_SelectedIndexChanged(object sender, EventArgs e)
{
    bool esteSelectat = lvCarti.SelectedItems.Count > 0;
    btnEditeaza.Enabled = esteSelectat;
    btnSterge.Enabled = esteSelectat;
}
```

Apelează această logică și în `Form1_Load` pentru starea inițială — când lista este goală sau nimic nu este selectat, butoanele trebuie să fie dezactivate de la pornire:

```csharp
private void Form1_Load(object sender, EventArgs e)
{
    lvCarti.Columns.Add("Titlu", 200, HorizontalAlignment.Left);
    // ... restul coloanelor ...

    btnEditeaza.Enabled = false;
    btnSterge.Enabled = false;

    RefreshLista();
}
```

### Evenimentul `DoubleClick`&#x20;

O îmbunătățire frecventă și apreciată de utilizatori: dublu-click pe un rând deschide direct formularul de editare, fără să mai fie nevoie să apese butonul „Editează". Se implementează reutilizând logica existentă din buton:

```csharp
private void lvCarti_DoubleClick(object sender, EventArgs e)
{
    // Reutilizam exact logica din butonul de editare
    // Nu duplicam cod — apelam handler-ul existent
    if (lvCarti.SelectedItems.Count > 0)
        btnEditeaza_Click(sender, EventArgs.Empty);
}
```


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-6-separarea-responsabilitatilor-listview/listview-in-modul-details.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
