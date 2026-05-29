# Desenarea cu Graphics

`Graphics` este clasa centrală a subsistemului grafic GDI+ din .NET. Prin ea se fac toate operațiile de desen: text, forme geometrice, imagini, curbe. Nu instanțiezi `Graphics` direct — primești un obiect gata creat fie din `e.Graphics` în `PrintPage`, fie din `CreateGraphics()` pe un control, fie dintr-un `Bitmap`.

Toate coordonatele sunt relative la colțul stânga-sus al suprafeței de desenare. Pe o pagină de imprimat, `e.MarginBounds.Left` și `e.MarginBounds.Top` definesc originea recomandată — tot conținutul începe de acolo și crește spre dreapta și în jos.

### `DrawString` — desenarea textului

`DrawString` este metoda pentru afișarea textului. Forma de bază primește textul, fontul, pensula de culoare și coordonatele colțului stânga-sus:

```csharp
g.DrawString("Andrei Popescu", font, Brushes.Black, x, y);
```

**`Font`** se creează cu numele fontului, mărimea și stilul. Mărimea este în puncte tipografice (points):

```csharp
Font fontTitlu   = new Font("Arial", 14, FontStyle.Bold);
Font fontStudent = new Font("Arial", 11, FontStyle.Bold);
Font fontMaterie = new Font("Arial", 10, FontStyle.Regular);
```

**`Brushes`** oferă pensule predefinite pentru culorile standard: `Brushes.Black`, `Brushes.White`, `Brushes.DarkGray`. Pensula definește culoarea cu care se „vopsește" textul.

O observație practică: fonturile create cu `new Font(...)` alocă resurse GDI. În cod de producție ar trebui eliberate cu `Dispose()`. Pentru exercițiu, .NET le eliberează la finalizarea garbage collection, deci nu este critic.

### `FillRectangle` — fundaluri colorate

`FillRectangle` umple un dreptunghi cu o culoare. Este operația de bază pentru fundalul antetului unui tabel sau pentru alternarea culorilor rândurilor:

```csharp
// Fundal inchis pentru antetul tabelului
g.FillRectangle(Brushes.DarkGray,
    new RectangleF(x, y, colMaterie + colMedie, inaltimeRand));

// Alternare alb / gri deschis pentru randuri
Brush fundal = (i % 2 == 0) ? Brushes.White
    : new SolidBrush(Color.FromArgb(240, 240, 240));
g.FillRectangle(fundal, new RectangleF(x, y, latime, inaltimeRand));
```

`RectangleF` primește patru parametri: x, y, lățime, înălțime — toate valori `float`. Se distinge de `Rectangle` care folosește `int` — în contextul imprimării, coordonatele flotante oferă mai multă precizie.

**Ordinea operațiilor este critică.** Desenezi întotdeauna fundalul cu `FillRectangle` înainte de a desena textul cu `DrawString` deasupra lui. Dacă inversezi ordinea, fundalul acoperă textul și pagina pare goală în acea zonă:

```csharp
// CORECT: fundal, apoi text
g.FillRectangle(Brushes.DarkGray, new RectangleF(x, y, latime, inaltimeRand));
g.DrawString("Materie", font, Brushes.White, x + 4, y + 2);

// GRESIT: textul este acoperit de fundal
g.DrawString("Materie", font, Brushes.White, x + 4, y + 2);
g.FillRectangle(Brushes.DarkGray, new RectangleF(x, y, latime, inaltimeRand));
```

Micul offset `x + 4` și `y + 2` în `DrawString` creează un spațiu interior (padding) între marginea celulei și text — fără el, textul ar fi lipit de bordura celulei.

### `DrawLine` și `DrawRectangle` — linii și chenare

`DrawLine` desenează o linie dreaptă între două puncte. `DrawRectangle` desenează conturul unui dreptunghi. Ambele primesc un `Pen` care definește culoarea și grosimea liniei:

```csharp
// Linie orizontala de separare sub un rand
g.DrawLine(Pens.Gray, x, y + inaltimeRand, x + latimeTabel, y + inaltimeRand);

// Contur exterior al tabelului
g.DrawRectangle(Pens.Black, x, yStartTabel, latimeTabel, inaltimeTotala);
```

`Pens.Gray` și `Pens.Black` sunt pene predefinite cu grosimea de 1 unitate. O linie separatoare internă folosește `Pens.Gray`, conturul exterior `Pens.Black` — diferența vizuală este subtilă dar face tabelul mai lizibil, comunicând ierarhia vizuală a elementelor.

Conturul exterior se desenează **după** toate rândurile, nu înainte. Dacă l-ai desena la început, rândurile l-ar acoperi parțial la margini.

### Calcularea pozițiilor coloanelor

Tabelul de medii are două coloane. Lățimile sunt valori fixe, iar poziția X a fiecărei coloane se obține cumulând lățimile coloanelor precedente:

```csharp
float colMaterie = 150;   // latimea primei coloane
float colMedie   = 80;    // latimea celei de-a doua coloane

// Coloana 1 incepe la x
// Coloana 2 incepe la x + colMaterie
g.DrawString("Matematica", font, Brushes.Black, x + 4,              y + 2);
g.DrawString("8.50",       font, Brushes.Black, x + colMaterie + 4, y + 2);
```

Lățimea totală a tabelului este `colMaterie + colMedie`. Aceasta se folosește pentru liniile de separare și pentru conturul exterior:

```csharp
float latimeTabel = colMaterie + colMedie;

g.DrawLine(Pens.Gray, x, y + inaltimeRand, x + latimeTabel, y + inaltimeRand);
g.DrawRectangle(Pens.Black, x, yStart, latimeTabel, inaltimeTotala);
```

### Structura completă a unui tabel simplu

Combinând toate operațiile, un tabel cu antet și rânduri arată astfel:

```csharp
float colMaterie   = 150;
float colMedie     = 80;
float inaltimeRand = 18;
float yStartTabel  = y;  // retinem y-ul de inceput pentru conturul exterior

// Antet
g.FillRectangle(Brushes.DarkGray, x, y, colMaterie + colMedie, inaltimeRand);
g.DrawString("Materie", fontAntet, Brushes.White, x + 4,              y + 2);
g.DrawString("Medie",   fontAntet, Brushes.White, x + colMaterie + 4, y + 2);
y += inaltimeRand;

// Randuri de date
for (int i = 0; i < materii.Length; i++)
{
    g.DrawString(materii[i],              font, Brushes.Black, x + 4,              y + 2);
    g.DrawString(medii[i].ToString("F2"), font, Brushes.Black, x + colMaterie + 4, y + 2);
    g.DrawLine(Pens.Gray, x, y + inaltimeRand, x + colMaterie + colMedie, y + inaltimeRand);
    y += inaltimeRand;
}

// Contur exterior — desenat dupa toate randurile
float inaltimeTotala = inaltimeRand * (materii.Length + 1);  // +1 pentru antet
g.DrawRectangle(Pens.Black, x, yStartTabel, colMaterie + colMedie, inaltimeTotala);
```

Variabila `yStartTabel` stochează `y`-ul înainte de a desena antetul, pentru că `DrawRectangle` are nevoie de coordonata de sus a tabelului, nu de cea curentă de după ultimul rând.


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-10-printing/desenarea-cu-graphics.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
