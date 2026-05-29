# Candlestick chart

Graficul Candlestick — numit și „lumânare japoneză" — este reprezentarea standard pentru date bursiere. Spre deosebire de Line chart care afișează o singură valoare per perioadă, Candlestick afișează patru valori simultan pentru fiecare zi:

**Open** — prețul la care s-a deschis tranzacționarea. **High** — cel mai mare preț atins în cursul zilei. **Low** — cel mai mic preț atins în cursul zilei. **Close** — prețul la care s-a închis tranzacționarea.

Vizual, fiecare zi apare ca o formă compusă din două elemente. **Corpul** este dreptunghiul dintre `Open` și `Close` — dacă `Close > Open` ziua a fost pozitivă, dacă `Close < Open` ziua a fost negativă. **Fitilele** sunt liniile subțiri care se extind deasupra și dedesubt, reprezentând intervalul dintre `Low` și `High`.

```
       │   ← fitil superior (High)
    ┌──┴──┐
    │     │  ← corp (Open → Close)
    └──┬──┘
       │   ← fitil inferior (Low)
```

Această formă comunică dintr-o privire mai multă informație decât orice alt tip de grafic bursier: nu doar unde s-a terminat ziua, ci și unde a început, cât de mult a fluctuat și în ce direcție s-a mișcat.

### Ordinea parametrilor în `Points.AddXY`

Adăugarea unui punct Candlestick se face cu `Points.AddXY`, dar cu patru valori Y în loc de una. Ordinea corectă este **High, Low, Open, Close** — nu ordinea acronimului OHLC:

```csharp
series.Points.AddXY(
    dataX,      // valoarea X
    high,       // Y1 — maximul zilei
    low,        // Y2 — minimul zilei
    open,       // Y3 — pretul de deschidere
    close       // Y4 — pretul de inchidere
);
```

Aceasta este una dintre cele mai frecvente greșeli la Candlestick. Chart Control nu aruncă excepție dacă furnizezi parametrii în ordinea greșită — afișează pur și simplu candlestick-uri incorecte, unde înălțimea și adâncimea fitilelor sunt inversate. Graficul arată plauzibil dar datele sunt greșite.

### Candlestick chart complet

```csharp
private void IncarcaChartCandlestick(Companie companie)
{
    chartCandlestick.ChartAreas.Clear();
    chartCandlestick.Series.Clear();
    chartCandlestick.Titles.Clear();

    chartCandlestick.Titles.Add(companie.Nume + " — OHLC");

    ChartArea area = new ChartArea("candlestick");
    area.AxisX.Title = "Data";
    area.AxisX.LabelStyle.Angle = -45;
    area.AxisX.Interval = 3;
    area.AxisX.MajorGrid.LineColor = Color.LightGray;
    area.AxisY.Title = "Preț (RON)";
    area.AxisY.MajorGrid.LineColor = Color.LightGray;
    area.AxisY.LabelStyle.Format = "F2";
    chartCandlestick.ChartAreas.Add(area);

    Series series = new Series("OHLC");
    series.ChartType = SeriesChartType.Candlestick;
    series.ChartArea = "candlestick";
    series.Color = Color.Black;              // culoarea fitilelor
    series["PriceUpColor"] = "SeaGreen";    // corpul zilelor pozitive
    series["PriceDownColor"] = "Tomato";    // corpul zilelor negative
    series["PointWidth"] = "0.7";           // latimea corpului
    chartCandlestick.Series.Add(series);

    List<ZiBursiera> zile = FakeDatabase.Zile
        .Where(z => z.Simbol == companie.Simbol)
        .OrderBy(z => z.Data)
        .ToList();

    foreach (ZiBursiera zi in zile)
    {
        series.Points.AddXY(
            zi.Data.ToString("dd.MM"),
            (double)zi.High,    // Y1 — High
            (double)zi.Low,     // Y2 — Low
            (double)zi.Open,    // Y3 — Open
            (double)zi.Close    // Y4 — Close
        );
    }
}
```

### Atributele custom — `series["NumeAtribut"]`

Chart Control folosește un mecanism special pentru configurații specifice unor tipuri de grafice: **atribute custom**, setate prin indexer pe `Series`:

```csharp
series["PriceUpColor"] = "SeaGreen";
series["PriceDownColor"] = "Tomato";
series["PointWidth"] = "0.7";
```

`series["NumeAtribut"]` este o pereche cheie-valoare stocată ca string — Chart Control o parsează intern și o aplică graficului. Valorile nu sunt obiecte `Color` sau `double` — sunt string-uri. Această sintaxă apare aproape exclusiv la Candlestick și câteva alte tipuri speciale, nu la `Line` sau `Column`.

`PriceUpColor` și `PriceDownColor` controlează culoarea corpului lumânării — verde pentru zilele pozitive (`Close > Open`), roșu pentru zilele negative. Dacă nu le setezi, Chart Control folosește culorile implicite care pot fi greu de interpretat.

`PointWidth` controlează lățimea corpului relativ la spațiul disponibil per punct. Valoarea `"0.7"` înseamnă 70% din spațiu — un echilibru vizual plăcut. Valori sub `0.4` produc lumânări prea înguste, valori peste `0.9` le fac să se suprapună.

### Ajustarea scării axei Y

Implicit, Chart Control pornește scala axei Y de la zero. Pentru prețuri bursiere între, să zicem, 45 și 60 RON, aceasta înseamnă că toate lumânările apar strânse în treimea superioară a graficului, iar diferențele zilnice sunt aproape invizibile.

Soluția este să setezi manual limitele axei, cu un mic spațiu de respirație (padding) deasupra și dedesubt:

```csharp
double min = (double)zile.Min(z => z.Low);
double max = (double)zile.Max(z => z.High);
double padding = (max - min) * 0.05;  // 5% din intervalul total ca spatiu vizual

area.AxisY.Minimum = min - padding;
area.AxisY.Maximum = max + padding;
```

Aceasta face diferențele zilnice vizibile și graficul mult mai lizibil. Fără această ajustare, un grafic Candlestick cu prețuri relativ stabile arată ca o linie plată.

### Comparație între cele trei tipuri

| Aspect                     | Line              | Column              | Candlestick                      |
| -------------------------- | ----------------- | ------------------- | -------------------------------- |
| **Valori Y per punct**     | 1                 | 1                   | 4                                |
| **Ordinea parametrilor Y** | `(y)`             | `(y)`               | `(high, low, open, close)`       |
| **Culori condiționate**    | `DataPoint.Color` | `DataPoint.Color`   | `PriceUpColor`, `PriceDownColor` |
| **Scara axei Y**           | Auto              | Auto                | Ajustată manual                  |
| **Potrivit pentru**        | Tendințe continue | Comparații discrete | Date bursiere OHLC               |


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-8-chart-control/candlestick-chart.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
