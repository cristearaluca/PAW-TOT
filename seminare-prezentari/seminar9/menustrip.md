# MenuStrip

`MenuStrip` este controlul care adaugă o bară de meniu clasică în partea de sus a ferestrei — exact ca meniurile din Notepad, Paint sau orice altă aplicație Windows. Itemurile sunt organizate ierarhic: meniuri principale la primul nivel, submeniuri la click pe ele, eventual submeniuri ale submeniurilor.

Față de butoane plasate pe formular, meniurile organizează mai bine funcționalitățile care nu sunt folosite constant. Butoanele ocupă spațiu vizual permanent — meniurile rămân discrete până când utilizatorul are nevoie de ele. Operațiile de fișier (Deschide, Salvează) și operațiile pe colecție (Adaugă, Editează, Șterge) sunt candidați naturali pentru meniu.

### Adăugarea în Designer

`MenuStrip` se trage din Toolbox și se plasează pe formular. Se atașează automat la marginea de sus a ferestrei, deasupra oricărui alt control. Celelalte controale nu sunt acoperite de el — `MenuStrip` rezervă spațiu propriu în layoutul ferestrei.

> 📸 **SCREENSHOT:** Formular cu `MenuStrip` adăugat, vizibil în partea de sus. Arată bara de componente de sub formular cu `menuStrip1` listat, și câmpul `"Type Here"` evidențiat — locul unde se adaugă primul item.

Itemurile se adaugă direct în Designer: click pe `"Type Here"` și tastezi textul itemului. La apăsarea Enter sau la click pe `"Type Here"` de dedesubt, adaugi itemuri la același nivel sau pe nivelul următor. Nu există limită de niveluri, dar în practică mai mult de două niveluri devine greu de navigat.

### Structura ierarhică

Un meniu tipic pentru exercițiu arată astfel în Designer:

```
MenuStrip
 ├── Fișier
 │    ├── Deschide
 │    └── Salvează
 └── Intrare
      ├── Adaugă
      ├── Editează
      └── Șterge
```

Itemurile de la primul nivel (`Fișier`, `Intrare`) sunt `ToolStripMenuItem`-uri. La click pe ele se deschide submeniul. Itemurile din submeniu sunt tot `ToolStripMenuItem`-uri, dar cu handler-ele de `Click` pe care le implementezi tu.

### Redenumirea itemurilor

Ca la orice control, redenumești imediat după adăugare. Convenția pentru `ToolStripMenuItem`: prefixul `mni` urmat de un nume descriptiv:

```
mniDeschide, mniSalveaza
mniAdauga, mniEditeaza, mniSterge
```

Redenumirea se face din fereastra Properties cu itemul selectat în Designer — câmpul `(Name)`. Textul vizibil se schimbă prin proprietatea `Text`.

### Abonarea la evenimentul `Click`

Dublu-click pe un item în Designer generează automat handler-ul `Click` și îl abonează:

```csharp
private void mniDeschide_Click(object sender, EventArgs e)
{
    // logica pentru Deschide
}

private void mniSalveaza_Click(object sender, EventArgs e)
{
    // logica pentru Salvează
}
```

`sender` este `ToolStripMenuItem` care a fost apăsat. Dacă ai nevoie de el, faci cast:

```csharp
ToolStripMenuItem item = sender as ToolStripMenuItem;
```

### Separatori

Un separator este o linie orizontală care grupează vizual itemurile înrudite. Se adaugă din Designer: click dreapta pe un item → **Insert → Separator**, sau tragi `ToolStripSeparator` din Toolbox în meniu.

```
Fișier
 ├── Deschide
 ├── Salvează
 ├── ─────────  ← separator
 └── Ieșire
```

### Activarea și dezactivarea itemurilor

`Enabled = false` dezactivează un item — apare estompat și nu poate fi apăsat. Util pentru a împiedica operațiile de editare și ștergere când nimic nu este selectat:

```csharp
private void lvIntrari_SelectedIndexChanged(object sender, EventArgs e)
{
    bool esteSelectat = lvIntrari.SelectedItems.Count > 0;
    mniEditeaza.Enabled = esteSelectat;
    mniSterge.Enabled = esteSelectat;
}
```

Apelează această logică și în `Form1_Load` pentru starea inițială.


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-9-serializarea-si-deserializarea/menustrip.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
