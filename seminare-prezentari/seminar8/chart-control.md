# Chart control

`Chart` este un control WinForms pentru vizualizarea datelor sub formă de grafice. Face parte din biblioteca `System.Windows.Forms.DataVisualization.Charting`, inclusă în .NET Framework, dar care necesită adăugarea explicită a unei referințe în proiect înainte de a putea fi folosită.

Chart Control suportă peste 30 de tipuri de grafice — `Line`, `Bar`, `Column`, `Pie`, `Area`, `Candlestick`, `Stock` și altele. Toate tipurile folosesc aceeași structură internă și aceleași mecanisme de configurare, ceea ce înseamnă că odată ce înțelegi cum funcționează un tip, tranziția la altul este directă.

### Adăugarea referinței în proiect

Înainte de a putea folosi `Chart` — în Designer sau în cod — trebuie adăugată referința la biblioteca corespunzătoare. Fără ea, controlul nu apare în Toolbox și `using`-ul nu poate fi rezolvat.

**Pasul 1:** Click dreapta pe proiect în Solution Explorer → **Add → Reference**.

> 📸 **SCREENSHOT:** Solution Explorer cu click dreapta pe proiect, meniul contextual cu „Add → Reference" evidențiat.

**Pasul 2:** În fereastra Reference Manager, navighează la **Assemblies → Framework**, caută `System.Windows.Forms.DataVisualization`, bifează componenta și apasă **OK**.

> 📸 **SCREENSHOT:** Fereastra Reference Manager cu tab-ul „Assemblies → Framework" activ, câmpul de căutare conținând „DataVisualization" și componenta `System.Windows.Forms.DataVisualization` bifată.

**Pasul 3:** Adaugă `using` la începutul fișierului:

```csharp
using System.Windows.Forms.DataVisualization.Charting;
```

Odată referința adăugată, controlul `Chart` apare și în Toolbox, în categoria **Data**, și poate fi plasat pe formular ca orice alt control.

### Structura internă a unui Chart

Un control `Chart` are o ierarhie de obiecte cu roluri distincte. Înțelegerea acestei ierarhii este esențială — fără ea, configurarea pare arbitrară.

```
Chart
 ├── ChartAreas  (colectie de zone de desenare)
 │    └── ChartArea  (o zona cu axe si grila)
 │         ├── AxisX  (axa orizontala)
 │         └── AxisY  (axa verticala)
 └── Series  (colectie de serii de date)
      └── Series  (o serie — o linie, un set de bare etc.)
           └── Points  (punctele de date)
```

**`ChartArea`** este suprafața de desenare — dreptunghiul cu axe și grilă pe care sunt trasate datele. Un `Chart` poate conține mai multe `ChartArea`-uri, dar pentru exercițiu folosim una singură per grafic.

**`Series`** este un set de date vizualizat grafic. Tipul de grafic — `Line`, `Column`, `Candlestick` — se specifică pe `Series`, nu pe `ChartArea`. Un `ChartArea` poate conține mai multe `Series` simultan, de exemplu două linii pe aceleași axe.

**`Points`** este colecția de puncte de date dintr-o `Series`. Fiecare punct are o valoare X și una sau mai multe valori Y — Candlestick, de exemplu, are patru valori Y per punct.

### Eliminarea elementelor default și configurarea din cod

Un `Chart` nou creat din Designer vine cu o `ChartArea` numită `"ChartArea1"` și o `Series` numită `"Series1"` deja configurate cu valori implicite. Dacă lași aceste elemente și mai adaugi altele din cod, ajungi cu configurații mixte greu de urmărit.

Cel mai clar mod de lucru: la fiecare reîncărcare a graficului, ștergi tot și reconstruiești din cod:

```csharp
private void IncarcaChartPret(Companie companie)
{
    chart.ChartAreas.Clear();
    chart.Series.Clear();
    chart.Titles.Clear();

    // De aici, totul este construit explicit din cod
    ChartArea area = new ChartArea("pret");
    chart.ChartAreas.Add(area);

    Series series = new Series("Preț închidere");
    series.ChartType = SeriesChartType.Line;
    series.ChartArea = "pret";
    chart.Series.Add(series);
}
```

Șterge tot la fiecare apel înseamnă că nu trebuie să te gândești la starea anterioară a graficului — fiecare apel produce un grafic curat, configurat exact cum vrei.

### Proprietăți esențiale ale `ChartArea`

`ChartArea` controlează axele, grila și aspectul general al suprafeței de desenare:

```csharp
ChartArea area = new ChartArea("pret");

// Titlurile axelor
area.AxisX.Title = "Data";
area.AxisY.Title = "Preț (RON)";

// Grila — culoare discreta pentru a nu distrage de la date
area.AxisX.MajorGrid.LineColor = Color.LightGray;
area.AxisY.MajorGrid.LineColor = Color.LightGray;

// Rotatia etichetelor pe axa X — evita suprapunerea cand datele sunt dese
area.AxisX.LabelStyle.Angle = -45;

// Frecventa etichetelor — o eticheta la fiecare 3 puncte
area.AxisX.Interval = 3;

// Formatul valorilor de pe axa Y
area.AxisY.LabelStyle.Format = "F2";  // doua zecimale

chart.ChartAreas.Add(area);
```

`LabelStyle.Angle = -45` este una dintre setările pe care le vei folosi aproape întotdeauna pe date temporale. Fără ea, etichetele datelor se suprapun când sunt mai mult de câteva puncte.

### Proprietăți esențiale ale `Series`

`Series` controlează tipul graficului și aspectul vizual al datelor:

```csharp
Series series = new Series("Preț închidere");

// Tipul graficului — setat pe Series, nu pe ChartArea
series.ChartType = SeriesCartType.Line;

// ChartArea pe care este desenata aceasta serie
series.ChartArea = "pret";  // trebuie sa corespunda numelui ChartArea adaugate

// Culoarea si grosimea liniei
series.Color = Color.SteelBlue;
series.BorderWidth = 2;

// Marker vizibil pe fiecare punct de date
series.MarkerStyle = MarkerStyle.Circle;
series.MarkerSize = 5;

chart.Series.Add(series);
```

Asocierea seriei cu `ChartArea` se face prin `series.ChartArea = "numeleChartArea"`. Dacă numele nu corespunde unei `ChartArea` existente, graficul nu afișează nimic — fără mesaj de eroare.


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-8-chart-control/chart-control.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
