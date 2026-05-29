# Line & Bar charts

**Line chart** (`SeriesChartType.Line`) conectează punctele de date cu o linie continuă. Este tipul potrivit pentru date care evoluează în timp — prețuri, temperaturi, trafic — unde continuitatea dintre puncte are sens și interesează tendința generală, nu valoarea izolată a fiecărui punct.

**Column chart** (`SeriesChartType.Column`) afișează fiecare valoare ca o bară verticală. Spre deosebire de Line, bara nu implică continuitate — fiecare valoare este independentă, iar comparația vizuală dintre valori alăturate este scopul principal. Este tipul potrivit pentru volume zilnice, vânzări pe categorii, distribuții.

`SeriesChartType.Bar` este varianta orizontală a aceluiași tip de grafic. Alegerea dintre `Column` și `Bar` este în principal vizuală — în exercițiu folosim `Column` pentru volume, care este mai intuitiv pentru serii temporale.

### Adăugarea punctelor cu `Points.AddXY`

Metoda `Points.AddXY` adaugă un punct de date în serie. Primul argument este valoarea X — categoria sau data — al doilea este valoarea Y:

```csharp
series.Points.AddXY(valueX, valueY);
```

Un detaliu important: `Points.AddXY` acceptă `double` pentru valorile numerice, nu `decimal`. Clasa `ZiBursiera` stochează prețurile ca `decimal`, deci la adăugare este necesar castul explicit:

```csharp
series.Points.AddXY(zi.Data.ToString("dd.MM"), (double)zi.Close);
```

Dacă omiți castul, compilatorul acceptă codul — `decimal` se convertește implicit la `double` — dar poți pierde precizie la valori cu multe zecimale. Castul explicit comunică intenția și elimină orice ambiguitate.

### Line chart complet

```csharp
private void IncarcaChartPret(Companie companie)
{
    chartPret.ChartAreas.Clear();
    chartPret.Series.Clear();
    chartPret.Titles.Clear();

    chartPret.Titles.Add(companie.Nume + " — Evoluție preț închidere");

    ChartArea area = new ChartArea("pret");
    area.AxisX.Title = "Data";
    area.AxisX.LabelStyle.Angle = -45;
    area.AxisX.Interval = 3;
    area.AxisX.MajorGrid.LineColor = Color.LightGray;
    area.AxisY.Title = "Preț (RON)";
    area.AxisY.MajorGrid.LineColor = Color.LightGray;
    area.AxisY.LabelStyle.Format = "F2";
    chartPret.ChartAreas.Add(area);

    Series series = new Series("Preț închidere");
    series.ChartType = SeriesChartType.Line;
    series.ChartArea = "pret";
    series.Color = Color.SteelBlue;
    series.BorderWidth = 2;
    series.MarkerStyle = MarkerStyle.Circle;
    series.MarkerSize = 5;
    chartPret.Series.Add(series);

    List<ZiBursiera> zile = FakeDatabase.Zile
        .Where(z => z.Simbol == companie.Simbol)
        .OrderBy(z => z.Data)
        .ToList();

    foreach (ZiBursiera zi in zile)
        series.Points.AddXY(zi.Data.ToString("dd.MM"), (double)zi.Close);
}
```

`MarkerStyle = MarkerStyle.Circle` adaugă un punct mic pe fiecare valoare. Pe un Line chart cu date dese, marker-ii fac valorile individuale mai ușor de identificat vizual. Fără ei, linia este continuă dar nu se vede unde se află fiecare punct de date.

### Column chart complet

```csharp
private void IncarcaChartVolum(Companie companie)
{
    chartVolum.ChartAreas.Clear();
    chartVolum.Series.Clear();
    chartVolum.Titles.Clear();

    chartVolum.Titles.Add(companie.Nume + " — Volum tranzacționat");

    ChartArea area = new ChartArea("volum");
    area.AxisX.Title = "Data";
    area.AxisX.LabelStyle.Angle = -45;
    area.AxisX.Interval = 3;
    area.AxisX.MajorGrid.LineColor = Color.LightGray;
    area.AxisY.Title = "Volum (acțiuni)";
    area.AxisY.MajorGrid.LineColor = Color.LightGray;
    // Formatare cu separator de mii — "1,250,000" in loc de "1250000"
    area.AxisY.LabelStyle.Format = "#,##0";
    chartVolum.ChartAreas.Add(area);

    Series series = new Series("Volum");
    series.ChartType = SeriesChartType.Column;
    series.ChartArea = "volum";
    series.Color = Color.SteelBlue;
    series.BorderWidth = 0;  // elimina chenarul negru implicit de pe bare
    chartVolum.Series.Add(series);

    List<ZiBursiera> zile = FakeDatabase.Zile
        .Where(z => z.Simbol == companie.Simbol)
        .OrderBy(z => z.Data)
        .ToList();

    foreach (ZiBursiera zi in zile)
        series.Points.AddXY(zi.Data.ToString("dd.MM"), zi.Volum);
}
```

`LabelStyle.Format = "#,##0"` formatează numerele mari cu separator de mii. Fără formatare, axa Y afișează `1250000` — greu de citit. Cu formatare, afișează `1,250,000`.

### Colorarea condiționată a barelor

Toate barele din exemplul de mai sus au aceeași culoare. O îmbunătățire vizuală frecventă pentru date de volum: colorezi fiecare bară în funcție de direcția față de ziua anterioară — verde dacă volumul a crescut, roșu dacă a scăzut.

Când vrei să personalizezi fiecare punct individual, nu mai folosești `Points.AddXY` — creezi explicit câte un `DataPoint` pentru fiecare valoare:

```csharp
for (int i = 0; i < zile.Count; i++)
{
    ZiBursiera zi = zile[i];
    DataPoint point = new DataPoint();
    point.SetValueXY(zi.Data.ToString("dd.MM"), zi.Volum);

    // Prima zi nu are zi precedenta — culoare neutra
    if (i == 0)
        point.Color = Color.SteelBlue;
    else if (zi.Volum >= zile[i - 1].Volum)
        point.Color = Color.SeaGreen;
    else
        point.Color = Color.Tomato;

    series.Points.Add(point);
}
```

`DataPoint` este clasa unui punct individual din serie. Proprietatea `Color` setată pe un `DataPoint` suprascrie culoarea implicită a seriei pentru acel punct specific. `SetValueXY` face același lucru ca `Points.AddXY` — diferența este că avem referința la obiectul `DataPoint` pe care îl putem configura înainte de adăugare.

### Mai multe serii pe același `ChartArea`

Un `ChartArea` poate găzdui mai multe serii simultan. Aceasta este util când vrei să compari două seturi de date pe aceleași axe — de exemplu, prețul de deschidere și cel de închidere pe același grafic:

```csharp
Series seriesClose = new Series("Închidere");
seriesClose.ChartType = SeriesChartType.Line;
seriesClose.ChartArea = "pret";
seriesClose.Color = Color.SteelBlue;

Series seriesOpen = new Series("Deschidere");
seriesOpen.ChartType = SeriesChartType.Line;
seriesOpen.ChartArea = "pret";  // acelasi ChartArea ca seriesClose
seriesOpen.Color = Color.Orange;
seriesOpen.BorderDashStyle = ChartDashStyle.Dash;  // linie intrerupta

chartPret.Series.Add(seriesClose);
chartPret.Series.Add(seriesOpen);
```

Când există mai multe serii, Chart Control afișează automat o legendă cu numele lor. Poziția și aspectul legendei pot fi controlate prin colecția `Legends`:

```csharp
chartPret.Legends.Clear();
Legend legenda = new Legend("legenda");
legenda.Docking = Docking.Bottom;
chartPret.Legends.Add(legenda);
```


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-8-chart-control/line-and-bar-charts.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
