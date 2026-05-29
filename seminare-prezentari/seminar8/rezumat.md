# Rezumat

### `TabControl`

`TabControl` organizează conținutul în pagini suprapuse navigate prin tab-uri. Fiecare `TabPage` este un container independent.

| Concept                      | Ce înseamnă                                                                            |
| ---------------------------- | -------------------------------------------------------------------------------------- |
| **`TabPage`**                | O pagină individuală — container independent de controale                              |
| **`Text` pe `TabPage`**      | Eticheta vizibilă — separată de `Name`                                                 |
| **Activarea în Designer**    | Click pe eticheta tab-ului înainte de a adăuga controale — altfel aparțin formularului |
| **`SelectedIndex`**          | Indexul tab-ului activ (0-based)                                                       |
| **`SelectedTab`**            | Tab-ul activ ca obiect `TabPage`                                                       |
| **`SelectedIndexChanged`**   | Eveniment la schimbarea tab-ului — pentru încărcare leneșă                             |
| **`Dock = Fill` în cascadă** | `TabControl` umple fereastra → controlul din tab umple tab-ul                          |

### Chart Control

Referința `System.Windows.Forms.DataVisualization.Charting` trebuie adăugată din **Reference Manager → Assemblies → Framework**.

```
Chart
 ├── ChartAreas → ChartArea (suprafața cu axe)
 │                  ├── AxisX
 │                  └── AxisY
 └── Series → Series (tipul graficului se setează aici)
               └── Points
```

```csharp
// Structura de baza — aceeasi pentru orice tip de grafic
chart.ChartAreas.Clear();
chart.Series.Clear();
chart.Titles.Clear();

ChartArea area = new ChartArea("nume");
area.AxisX.Title = "...";
area.AxisX.LabelStyle.Angle = -45;
area.AxisX.Interval = 3;
area.AxisY.LabelStyle.Format = "F2";
area.AxisX.MajorGrid.LineColor = Color.LightGray;
area.AxisY.MajorGrid.LineColor = Color.LightGray;
chart.ChartAreas.Add(area);

Series series = new Series("Nume");
series.ChartType = SeriesChartType.Line;  // sau Column, Candlestick
series.ChartArea = "nume";               // trebuie sa corespunda exact
series.Color = Color.SteelBlue;
chart.Series.Add(series);
```

| Concept                              | Ce înseamnă                                                             |
| ------------------------------------ | ----------------------------------------------------------------------- |
| **`ChartArea`**                      | Suprafața de desenare cu axe și grilă                                   |
| **`series.ChartArea`**               | Asocierea seriei cu `ChartArea` după nume — trebuie să corespundă exact |
| **`Clear()` la fiecare reîncărcare** | Șterge tot și reconstruiești de la zero — fără stări mixte              |
| **`LabelStyle.Angle = -45`**         | Rotirea etichetelor — esențial pentru date temporale dese               |

### Line chart și Column chart

```csharp
// Line
series.ChartType = SeriesChartType.Line;
series.BorderWidth = 2;
series.MarkerStyle = MarkerStyle.Circle;
series.MarkerSize = 5;

// Column
series.ChartType = SeriesChartType.Column;
series.BorderWidth = 0;  // elimina chenarul negru implicit

// Adaugare puncte — Y trebuie double, nu decimal
series.Points.AddXY(zi.Data.ToString("dd.MM"), (double)zi.Close);

// Colorare conditionata per bara — cu DataPoint explicit
DataPoint point = new DataPoint();
point.SetValueXY(zi.Data.ToString("dd.MM"), zi.Volum);
point.Color = zi.Volum >= ziPrecedenta.Volum ? Color.SeaGreen : Color.Tomato;
series.Points.Add(point);
```

| Concept                | Ce înseamnă                                                    |
| ---------------------- | -------------------------------------------------------------- |
| **`(double)zi.Close`** | Cast explicit necesar — `AddXY` acceptă `double`, nu `decimal` |
| **`"#,##0"`**          | Format pentru numere mari cu separator de mii                  |
| **`DataPoint`**        | Punct individual — pentru culoare sau stil diferit per punct   |
| **`SetValueXY`**       | Echivalentul `AddXY` pe un `DataPoint` explicit                |

### Candlestick chart

```csharp
series.ChartType = SeriesChartType.Candlestick;
series.Color = Color.Black;              // fitilele
series["PriceUpColor"] = "SeaGreen";    // corp zi pozitiva — string, nu Color
series["PriceDownColor"] = "Tomato";    // corp zi negativa — string, nu Color
series["PointWidth"] = "0.7";

// Ordinea corecta: X, High, Low, Open, Close — nu OHLC
series.Points.AddXY(
    zi.Data.ToString("dd.MM"),
    (double)zi.High,
    (double)zi.Low,
    (double)zi.Open,
    (double)zi.Close
);

// Ajustarea scarii Y — obligatorie pentru prețuri în interval îngust
double min = (double)zile.Min(z => z.Low);
double max = (double)zile.Max(z => z.High);
double padding = (max - min) * 0.05;
area.AxisY.Minimum = min - padding;
area.AxisY.Maximum = max + padding;
```

| Concept                         | Ce înseamnă                                                                     |
| ------------------------------- | ------------------------------------------------------------------------------- |
| **Ordinea Y**                   | `(high, low, open, close)` — greșeala nu produce eroare, produce date incorecte |
| **`series["atribut"]`**         | Atribute custom — string-uri, nu obiecte .NET                                   |
| **`AxisY.Minimum` / `Maximum`** | Fără ajustare, prețurile în interval îngust apar ca o linie                     |


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-8-chart-control/rezumat.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
