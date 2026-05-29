# Rezumat

### 1. Aplicații MDI

MDI (Multiple Document Interface) — un container care găzduiește ferestre copil în interior. Ferestrele copil nu sunt independente față de sistemul de operare — nu au intrare separată în taskbar și sunt limitate la spațiul containerului.

| Concept                     | Ce înseamnă                                                               |
| --------------------------- | ------------------------------------------------------------------------- |
| **`IsMdiContainer = true`** | Transformă `Form`-ul în container MDI                                     |
| **`f.MdiParent = this`**    | Înregistrează fereastra copil în container — înainte de `Show()`          |
| **`MdiChildren`**           | Colecția ferestrelor copil deschise curent                                |
| **`f.Activate()`**          | Aduce o fereastră copil existentă în prim-plan                            |
| **`LayoutMdi`**             | Aranjează automat ferestrele: `Cascade`, `TileHorizontal`, `TileVertical` |

```csharp
// Deschidere fereastra copil — verificare daca exista deja
private void mniListaTaskuri_Click(object sender, EventArgs e)
{
    foreach (Form f in this.MdiChildren)
    {
        if (f is FormLista)
        {
            f.Activate();
            return;
        }
    }

    FormLista lista = new FormLista();
    lista.MdiParent = this;
    lista.Show();
}
```

### 2. `ContextMenuStrip`

Meniu apărut la click dreapta — oferă acțiuni relevante pentru elementul selectat. **Nu** se asociază prin `control.ContextMenuStrip` — aceasta l-ar afișa și pe zonele goale. Se afișează manual prin `MouseClick` + `HitTest`.

| Concept                               | Ce înseamnă                                                                       |
| ------------------------------------- | --------------------------------------------------------------------------------- |
| **`HitTest(x, y)`**                   | Testează ce item se află la coordonatele click-ului — `Item == null` = zonă goală |
| **`cmsTaskuri.Show(control, point)`** | Afișează meniul manual la coordonatele mouse-ului                                 |
| **`info.Item.Selected = true`**       | Selectează itemul la click dreapta — pentru coerență vizuală                      |
| **`Opening`**                         | Eveniment înainte de afișare — activezi/dezactivezi itemurile                     |
| **`e.Cancel = true`**                 | Anulează afișarea meniului                                                        |
| **`item.Enabled`**                    | Dezactivează un item — vizibil, estompat, inaccesibil                             |
| **`SelectedItems[0].Tag`**            | Recuperează obiectul asociat rândului selectat                                    |

```csharp
// MouseClick — afisare meniu doar pe item, nu pe zona goala
private void lvTaskuri_MouseClick(object sender, MouseEventArgs e)
{
    if (e.Button != MouseButtons.Right) return;

    ListViewHitTestInfo info = lvTaskuri.HitTest(e.X, e.Y);
    if (info.Item == null) return;

    info.Item.Selected = true;
    cmsTaskuri.Show(lvTaskuri, e.Location);
}

// Opening — configureaza itemurile inainte de afisare
private void cmsTaskuri_Opening(object sender, CancelEventArgs e)
{
    if (lvTaskuri.SelectedItems.Count == 0) { e.Cancel = true; return; }

    TodoTask task = lvTaskuri.SelectedItems[0].Tag as TodoTask;
    mniMarcheaza.Enabled  = !task.Efectuat;
    mniRedeschide.Enabled =  task.Efectuat;
}

// Handler-ele itemurilor
private void mniMarcheaza_Click(object sender, EventArgs e)
{
    (lvTaskuri.SelectedItems[0].Tag as TodoTask).Efectuat = true;
    RefreshLista();
}

private void mniRedeschide_Click(object sender, EventArgs e)
{
    (lvTaskuri.SelectedItems[0].Tag as TodoTask).Efectuat = false;
    RefreshLista();
}

private void mniSterge_Click(object sender, EventArgs e)
{
    TodoTask task = lvTaskuri.SelectedItems[0].Tag as TodoTask;

    if (MessageBox.Show("Ștergi \"" + task.Titlu + "\"?", "Confirmare",
        MessageBoxButtons.YesNo, MessageBoxIcon.Question) == DialogResult.Yes)
    {
        FakeDatabase.Taskuri.Remove(task);
        RefreshLista();
    }
}
```


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-11-aplicatii-mdi/rezumat.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
