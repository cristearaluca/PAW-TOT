# Rezumat

### `PrintDocument` și `PrintPage`

Imprimarea în WinForms înseamnă desenarea explicită a conținutului paginii prin instrucțiuni GDI+. `PrintDocument` este componenta care gestionează comunicarea cu sistemul de imprimare. `PrintPage` este evenimentul declanșat pentru fiecare pagină — tot codul de desenare se află în handler-ul acestui eveniment.

| Concept              | Ce înseamnă                                                              |
| -------------------- | ------------------------------------------------------------------------ |
| **`PrintDocument`**  | Componentă din Toolbox → Printing — nu are reprezentare vizuală          |
| **`PrintPage`**      | Eveniment declanșat per pagină — tot desenul se face aici                |
| **`e.Graphics`**     | Obiectul cu care desenezi — același API GDI+ ca la controale             |
| **`e.MarginBounds`** | Zona utilizabilă a paginii — `Left` și `Top` sunt originea recomandată   |
| **`e.HasMorePages`** | `true` pentru documente multipagină — `PrintPage` se declanșează din nou |
| **Sutimi de inch**   | Unitatea de măsură în contextul imprimării                               |

```csharp
private void printDocument1_PrintPage(object sender, PrintPageEventArgs e)
{
    Graphics g = e.Graphics;
    float x = e.MarginBounds.Left;
    float y = e.MarginBounds.Top;

    // Titlu
    g.DrawString("Catalog note studenti", fontTitlu, Brushes.Black, x, y);
    y += 40;

    // Lista studentilor cu materiile indentate
    foreach (Student s in FakeDatabase.Studenti)
    {
        g.DrawString(s.Nume, fontStudent, Brushes.Black, x, y);      y += 20;
        g.DrawString("Matematica: " + s.Matematica, font, Brushes.Black, x + 30, y); y += 16;
        // ...
        y += 8;
    }

    // Tabel de medii
    // ...
}
```

**Tehnica fundamentală:** `y += inaltimeElement` după fiecare element avanseaza cursorul pe pagină. Indentarea = `x + 30`.

### Desenarea cu `Graphics`

| Metodă                                    | Ce face                                                      |
| ----------------------------------------- | ------------------------------------------------------------ |
| **`DrawString(text, font, brush, x, y)`** | Text la coordonate absolute                                  |
| **`FillRectangle(brush, RectangleF)`**    | Fundal colorat — pentru antet și alternarea rândurilor       |
| **`DrawLine(pen, x1, y1, x2, y2)`**       | Linie — separatori interni ai tabelului                      |
| **`DrawRectangle(pen, x, y, w, h)`**      | Conturul exterior al tabelului                               |
| **`Brushes.Culoare`**                     | Pensule predefinite — pentru `DrawString` și `FillRectangle` |
| **`Pens.Culoare`**                        | Pene predefinite — pentru `DrawLine` și `DrawRectangle`      |
| **`RectangleF(x, y, w, h)`**              | Dreptunghi cu coordonate `float`                             |

**Ordinea per rând de tabel:** `FillRectangle` → `DrawString` → `DrawLine`. Conturul exterior cu `DrawRectangle` se desenează ultimul — după toate rândurile.

```csharp
float colMaterie   = 150;
float colMedie     = 80;
float inaltimeRand = 18;
float yStartTabel  = y;  // retinem pentru DrawRectangle

// Antet
g.FillRectangle(Brushes.DarkGray, x, y, colMaterie + colMedie, inaltimeRand);
g.DrawString("Materie", fontAntet, Brushes.White, x + 4, y + 2);
g.DrawString("Medie",   fontAntet, Brushes.White, x + colMaterie + 4, y + 2);
y += inaltimeRand;

// Randuri
for (int i = 0; i < materii.Length; i++)
{
    g.DrawString(materii[i],              font, Brushes.Black, x + 4,              y + 2);
    g.DrawString(medii[i].ToString("F2"), font, Brushes.Black, x + colMaterie + 4, y + 2);
    g.DrawLine(Pens.Gray, x, y + inaltimeRand, x + colMaterie + colMedie, y + inaltimeRand);
    y += inaltimeRand;
}

// Contur exterior — dupa toate randurile
g.DrawRectangle(Pens.Black, x, yStartTabel,
    colMaterie + colMedie, inaltimeRand * (materii.Length + 1));
```

`yStartTabel` este necesar deoarece `DrawRectangle` are nevoie de coordonata de sus a tabelului, nu de cea curentă după ultimul rând.

### `PrintPreviewDialog`

| Concept                   | Ce înseamnă                                                            |
| ------------------------- | ---------------------------------------------------------------------- |
| **`preview.Document`**    | Legătura cu `PrintDocument` — obligatoriu, altfel excepție             |
| **`ShowDialog()`**        | Deschide fereastra și declanșează `PrintPage`                          |
| **Toolbar integrat**      | Zoom, navigare pagini, imprimare — gata implementate                   |
| **Același cod**           | `PrintPage` rulează identic pentru preview și `printDocument1.Print()` |
| **`PrintPreviewControl`** | Alternativa pentru integrarea în formular propriu                      |

```csharp
using (PrintPreviewDialog preview = new PrintPreviewDialog())
{
    preview.Document = printDocument1;
    preview.Width    = 800;
    preview.Height   = 650;
    preview.Text     = "Previzualizare catalog note";
    preview.ShowDialog();
}
```


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-10-printing/rezumat.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
