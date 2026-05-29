# ContextMenuStrip

Un **meniu contextual** (sau meniu de context) este un meniu care apare la click dreapta pe un element, oferind acțiuni relevante pentru acel element specific. Este diferit de meniul principal al aplicației — meniul principal oferă acțiuni globale, meniul contextual oferă acțiuni legate de exact ceea ce a selectat utilizatorul.

În exercițiu, click dreapta pe un task din `ListView` afișează opțiunile „Marchează ca efectuat", „Redeschide" și „Șterge" — acțiuni care au sens exclusiv în contextul task-ului selectat.

### Adăugarea `ContextMenuStrip` în Designer

`ContextMenuStrip` se adaugă din Toolbox exact ca `MenuStrip` — apare în bara de componente sub formular, nu pe suprafața lui. Itemurile se adaugă direct în Designer: click pe `"Type Here"` și tastezi textul.

> 📸 **SCREENSHOT:** Designer-ul cu `ContextMenuStrip` selectat în bara de componente și itemurile „Marchează ca efectuat", „Redeschide", „Șterge" vizibile. Arată că se editează direct în Designer, similar cu `MenuStrip`.

Redenumești itemurile imediat după adăugare — convenția `mni` se aplică și aici: `mniMarcheaza`, `mniRedeschide`, `mniSterge`.

### Asocierea prin proprietate vs. afișare manuală

Modul cel mai simplu de a lega un `ContextMenuStrip` de un control este prin proprietatea `ContextMenuStrip` a controlului:

```csharp
lvTaskuri.ContextMenuStrip = cmsTaskuri;
```

Aceasta face ca meniul să apară la orice click dreapta pe `ListView` — inclusiv pe zonele goale, fără niciun item selectat. Acesta este comportamentul nedorit: nu vrem să afișăm opțiuni de editare și ștergere când utilizatorul a dat click pe spațiul gol din listă.

Soluția este să **nu** atribuim `ContextMenuStrip` controlului prin proprietate, ci să gestionăm manual afișarea meniului prin evenimentul `MouseClick` al `ListView`-ului:

```csharp
// Nu setam lvTaskuri.ContextMenuStrip — lasam null
// In schimb, ascultam MouseClick si decidem noi cand se afiseaza meniul

private void lvTaskuri_MouseClick(object sender, MouseEventArgs e)
{
    if (e.Button != MouseButtons.Right) return;

    // HitTest returneaza informatii despre ce s-a dat click
    ListViewHitTestInfo info = lvTaskuri.HitTest(e.X, e.Y);

    // Afisam meniul doar daca s-a dat click pe un item real
    if (info.Item == null) return;

    // Selectam itemul pe care s-a dat click dreapta
    info.Item.Selected = true;

    // Afisam meniul la coordonatele mouse-ului
    cmsTaskuri.Show(lvTaskuri, e.Location);
}
```

`HitTest(x, y)` testează ce se găsește la coordonatele date în interiorul `ListView`-ului. Returnează un `ListViewHitTestInfo` cu proprietatea `Item` — care este `null` dacă click-ul a căzut pe o zonă goală sau pe antet, și conține `ListViewItem`-ul dacă click-ul a căzut pe un rând de date.

`info.Item.Selected = true` selectează explicit itemul pe care s-a dat click dreapta. Fără această linie, itemul rămâne neselectat vizual chiar dacă meniul se deschide deasupra lui — comportament confuz pentru utilizator.

`cmsTaskuri.Show(lvTaskuri, e.Location)` afișează meniul la coordonatele mouse-ului, relativ la `lvTaskuri`. `e.Location` este un `Point` cu `(e.X, e.Y)`.

### Configurarea itemurilor înainte de afișare — evenimentul `Opening`

Chiar și cu afișarea manuală, evenimentul `Opening` al `ContextMenuStrip`-ului se declanșează imediat înainte ca meniul să devină vizibil — atât la afișarea prin proprietate, cât și la cea manuală cu `Show()`. Este locul potrivit pentru a activa sau dezactiva itemurile în funcție de starea task-ului selectat:

```csharp
private void cmsTaskuri_Opening(object sender, CancelEventArgs e)
{
    if (lvTaskuri.SelectedItems.Count == 0)
    {
        e.Cancel = true;
        return;
    }

    TodoTask task = lvTaskuri.SelectedItems[0].Tag as TodoTask;

    mniMarcheaza.Enabled  = !task.Efectuat;
    mniRedeschide.Enabled =  task.Efectuat;
}
```

Deoarece în `MouseClick` am verificat deja că există un item și l-am selectat, verificarea `SelectedItems.Count == 0` din `Opening` este un strat suplimentar de siguranță — bună practică pentru a nu presupune o stare.

Dezactivarea itemurilor irelevante comunică vizual ce acțiuni sunt posibile în contextul curent. Un task deja efectuat nu mai poate fi „marcat ca efectuat" — itemul apare estompat și nu poate fi apăsat. Itemurile rămân vizibile pentru că utilizatorul trebuie să știe că opțiunea există, chiar dacă nu este disponibilă acum.

### Handler-ele itemurilor

Fiecare item din `ContextMenuStrip` are propriul eveniment `Click`, abonat din Designer cu dublu-click:

```csharp
private void mniMarcheaza_Click(object sender, EventArgs e)
{
    TodoTask task = lvTaskuri.SelectedItems[0].Tag as TodoTask;
    task.Efectuat = true;
    RefreshLista();
}

private void mniRedeschide_Click(object sender, EventArgs e)
{
    TodoTask task = lvTaskuri.SelectedItems[0].Tag as TodoTask;
    task.Efectuat = false;
    RefreshLista();
}

private void mniSterge_Click(object sender, EventArgs e)
{
    TodoTask task = lvTaskuri.SelectedItems[0].Tag as TodoTask;

    DialogResult raspuns = MessageBox.Show(
        "Ștergi task-ul \"" + task.Titlu + "\"?",
        "Confirmare ștergere",
        MessageBoxButtons.YesNo,
        MessageBoxIcon.Question);

    if (raspuns == DialogResult.Yes)
    {
        FakeDatabase.Taskuri.Remove(task);
        RefreshLista();
    }
}
```

Deoarece `MouseClick` a verificat că există un item și l-a selectat, în handler-ele itemurilor putem accesa direct `SelectedItems[0]` fără verificare suplimentară.

### Afișarea vizuală a task-urilor efectuate

O îmbunătățire de UX în `RefreshLista()`: task-urile efectuate se diferențiază vizual prin culoare gri și stil italic, comunicând imediat starea fără să fie nevoie să citești coloana „Status":

```csharp
private void RefreshLista()
{
    lvTaskuri.Items.Clear();

    foreach (TodoTask t in FakeDatabase.Taskuri)
    {
        ListViewItem item = new ListViewItem(t.Id.ToString());
        item.SubItems.Add(t.Titlu);
        item.SubItems.Add(t.Prioritate.ToString());
        item.SubItems.Add(t.DataCreare.ToString("dd.MM.yyyy"));
        item.SubItems.Add(t.Efectuat ? "Efectuat" : "Deschis");
        item.Tag = t;

        if (t.Efectuat)
        {
            item.ForeColor = Color.Gray;
            item.Font = new Font(lvTaskuri.Font, FontStyle.Italic);
        }

        lvTaskuri.Items.Add(item);
    }
}
```

### Separatori în `ContextMenuStrip`

Un separator este o linie orizontală care grupează vizual itemurile înrudite. Se adaugă din Designer cu click dreapta pe un item → **Insert Separator**. Un separator între „Redeschide" și „Șterge" separă vizual acțiunile de schimbare a stării de acțiunea distructivă de ștergere:

```
Marchează ca efectuat
Redeschide
──────────────────
Șterge
```


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-11-aplicatii-mdi/contextmenustrip.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
