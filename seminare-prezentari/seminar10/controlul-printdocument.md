# Controlul PrintDocument

Imprimarea în WinForms nu înseamnă să trimiți un fișier la imprimantă și să speri că arată bine. Înseamnă să **desenezi** conținutul paginii cu instrucțiuni grafice explicite — exact la fel cum ai desena pe un `Panel` sau un `PictureBox`, dar pe o suprafață care reprezintă o coală de hârtie.

Această abordare are un avantaj important față de alternativele care produc un document intermediar (PDF, HTML, RTF): tot ce desenezi apare exact cum l-ai specificat, fără conversii sau interpretări. Dezavantajul este că tu ești responsabil pentru fiecare element de pe pagină — nu există un motor de layout care să calculeze automat pozițiile.

Mecanismul central este componenta `PrintDocument`. Când i se cere să imprime — fie pentru o imprimantă reală, fie pentru previzualizare — declanșează evenimentul `PrintPage` pentru fiecare pagină a documentului. Tu te abonezi la acel eveniment și desenezi tot conținutul paginii prin obiectul `e.Graphics` primit ca argument.

### Adăugarea `PrintDocument` în proiect

`PrintDocument` nu este un control vizibil — este o **componentă**. Se adaugă din Toolbox, categoria **Printing**, și apare în bara de componente de sub formular, nu pe suprafața lui. Această distincție reflectă faptul că `PrintDocument` nu are reprezentare vizuală proprie — este un obiect care gestionează comunicarea cu sistemul de imprimare al Windows.

> 📸 **SCREENSHOT:** Toolbox cu categoria „Printing" expandată, componentele `PrintDocument`, `PrintPreviewDialog` și `PrintDialog` vizibile. Alături, bara de componente sub formular cu `printDocument1` adăugat — arată că apare sub formular, nu pe suprafața lui.

Abonarea la evenimentul `PrintPage` se face din fereastra Properties cu tab-ul de evenimente activ (iconița cu fulger), dublu-click pe `PrintPage`. Visual Studio generează automat handler-ul în `Form1.cs` și abonarea în `Form1.Designer.cs`:

```csharp
// In Form1.Designer.cs — generat automat
this.printDocument1.PrintPage +=
    new System.Drawing.Printing.PrintPageEventHandler(this.printDocument1_PrintPage);

// In Form1.cs — completat de tine
private void printDocument1_PrintPage(object sender, PrintPageEventArgs e)
{
    // Tot codul de desenare al paginii se afla aici
}
```

### `PrintPageEventArgs` — ce primești în handler

Argumentul `e` de tip `PrintPageEventArgs` conține tot ce ai nevoie pentru a desena pagina.

**`e.Graphics`** este obiectul `Graphics` pe care desenezi. Este exact același tip de obiect cu care lucrezi la orice operație grafică GDI+ în WinForms — dacă știi să desenezi pe un `Panel`, știi să desenezi și pe pagina de imprimat. Diferența este că, în contextul imprimării, coordonatele sunt în **sutimi de inch**, nu în pixeli.

**`e.MarginBounds`** este un `Rectangle` care descrie zona utilizabilă a paginii după scăderea marginilor implicite. Coordonatele `Left` și `Top` sunt punctul de start recomandat pentru conținut — nu `(0, 0)`, care reprezintă colțul fizic al paginii înainte de margini. O pagină A4 cu margini de un inch are `MarginBounds.Left ≈ 100` și `MarginBounds.Top ≈ 100`.

**`e.PageBounds`** este întreaga suprafață a paginii, inclusiv marginile. În general lucrezi cu `MarginBounds`.

**`e.HasMorePages`** este o proprietate pe care o setezi pe `true` dacă ai mai mult conținut decât încape pe o pagină și vrei ca `PrintPage` să fie declanșat din nou pentru pagina următoare. Gestionarea paginilor multiple implică să ții minte unde ai rămas în colecția de date între apeluri. În exercițiu presupunem că totul încape pe o pagină.

```csharp
private void printDocument1_PrintPage(object sender, PrintPageEventArgs e)
{
    Graphics g = e.Graphics;
    float x = e.MarginBounds.Left;   // marginea stanga — punctul de start X
    float y = e.MarginBounds.Top;    // marginea de sus — punctul de start Y

    // Desenam incepand din (x, y) si avansam y dupa fiecare element
    e.HasMorePages = false;  // implicit false, nu e nevoie sa il setezi explicit
}
```

### Unitatea de măsură — sutimi de inch

`e.Graphics` în contextul imprimării folosește **sutimi de inch** ca unitate implicită, nu pixeli. O pagină A4 are aproximativ 827 × 1169 sutimi de inch. `e.MarginBounds` returnează coordonate în această unitate.

Aceasta înseamnă că valorile pe care le specifici pentru coordonate, dimensiuni de fonturi și lățimi sunt în sutimi de inch. Un font de `10` puncte și o înălțime de rând de `18` sutimi de inch produc un text lizibil și bine spațiat. Nu trebuie să faci conversii manuale — lucrezi direct cu valorile și ajustezi vizual în previzualizare.

### Logica de avansare pe pagină

Toată poziționarea conținutului pe pagină se reduce la avansarea coordonatei `y` după fiecare element desenat. Desenezi un element la coordonata `y` curentă, adaugi înălțimea elementului la `y`, desenezi următorul la noua valoare. Aceasta este tehnica fundamentală pentru orice raport secvențial:

```csharp
// Titlu
g.DrawString("Catalog note studenti", fontTitlu, Brushes.Black, x, y);
y += 40;  // inaltimea titlului (35) + spatiu de respiratie (5)

// Primul student
g.DrawString("Andrei Popescu", fontStudent, Brushes.Black, x, y);
y += 20;  // inaltimea numelui + mic spatiu

// Materiile, indentate cu 30 de unitati fata de marginea stanga
g.DrawString("Matematica:  9", fontMaterie, Brushes.Black, x + 30, y);
y += 16;  // inaltimea unei linii de materie
```

**Indentarea** se obține simplu: `x + 30` mută textul cu 30 de unități la dreapta față de marginea stângă. Există un singur nivel de indentare — numele studentului la `x`, materiile la `x + 30`.

### De ce `PrintPage` și nu o altă abordare

Ai putea genera un document text simplu sau un fișier HTML și să îl deschizi cu o aplicație externă. `PrintPage` cu GDI+ are avantaje clare față de aceste alternative: nu depinde de aplicații externe, controlul asupra aspectului este complet și precis, și același cod funcționează atât pentru previzualizare cât și pentru imprimarea efectivă. Dezavantajul — că trebuie să calculezi manual pozițiile — devine neglijabil pentru rapoarte simple și secvențiale cum este catalogul de note.


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-10-printing/controlul-printdocument.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
