# Exercițiu

Construiești o aplicație care afișează notele studenților într-un `DataGridView` și permite previzualizarea unui raport înainte de imprimare. La apăsarea butonului de previzualizare, se deschide `PrintPreviewDialog` cu un raport care listează fiecare student cu notele sale, urmat de un tabel cu mediile generale per materie.

### Modelul de date și `FakeDatabase`&#x20;

Declară clasa `Student` cu proprietățile:

* `string Nume`
* `int Matematica`
* `int Informatica`
* `int Fizica`
* `int Chimie`
* `int Biologie`

Declară clasa statică `FakeDatabase` cu `static List<Student> Studenti` populată cu cel puțin 5 studenți cu note între 4 și 10.

### Fereastra principală

Pe `Form1` adaugă:

* Un `DataGridView` (`dgvStudenti`) cu `Dock = Fill` sau ancorat să ocupe majoritatea ferestrei
* Un `Button` (`btnPrintPreview`) cu textul `"Previzualizare"`
* Componenta `PrintDocument` (`printDocument1`) din Toolbox → Printing

În `Form1_Load`, setează `dgvStudenti.DataSource = FakeDatabase.Studenti`.

Abonează-te la evenimentul `PrintPage` al lui `printDocument1`.

### Raportul text cu lista studenților

În handler-ul `PrintPage`, desenează:

**Titlul raportului** — font mare, bold, la coordonatele `(x, y)`:

```csharp
g.DrawString("Catalog note studenti", fontTitlu, Brushes.Black, x, y);
y += 40;
```

**Lista studenților** — pentru fiecare student, numele urmat de materiile indentate:

```csharp
foreach (Student s in FakeDatabase.Studenti)
{
    g.DrawString(s.Nume, fontStudent, Brushes.Black, x, y);
    y += 20;

    g.DrawString("Matematica:  " + s.Matematica, fontMaterie, Brushes.Black, x + 30, y);
    y += 16;
    // ... celelalte materii
    y += 8;  // spatiu suplimentar intre studenti
}
```

Indentarea se obține folosind `x + 30` în loc de `x` pentru materiile din cadrul fiecărui student.

### Tabelul cu medii

La finalul listei de studenți, adaugă un titlu `"Medii generale"` și un tabel cu două coloane: `Materie` și `Medie`.

Calculează mediile înainte de a desena:

```csharp
double medMatematica = 0;
foreach (Student s in FakeDatabase.Studenti)
    medMatematica += s.Matematica;
medMatematica /= FakeDatabase.Studenti.Count;
// ... la fel pentru celelalte materii
```

Tabelul conține:

* Un antet cu `FillRectangle` colorat și `DrawString` alb
* Un rând per materie cu `DrawString` și o linie orizontală de separare cu `DrawLine`
* Un chenar exterior cu `DrawRectangle`

### Deschiderea `PrintPreviewDialog`

La click pe `btnPrintPreview`:

```csharp
using (PrintPreviewDialog preview = new PrintPreviewDialog())
{
    preview.Document = printDocument1;
    preview.Width = 800;
    preview.Height = 650;
    preview.ShowDialog();
}
```

### Date de test

```csharp
public static List<Student> Studenti = new List<Student>
{
    new Student { Nume = "Andrei Popescu",   Matematica = 9,  Informatica = 10, Fizica = 8, Chimie = 7,  Biologie = 9  },
    new Student { Nume = "Maria Ionescu",    Matematica = 7,  Informatica = 8,  Fizica = 6, Chimie = 9,  Biologie = 8  },
    new Student { Nume = "Alexandru Marin",  Matematica = 5,  Informatica = 7,  Fizica = 6, Chimie = 5,  Biologie = 6  },
    new Student { Nume = "Elena Constantin", Matematica = 10, Informatica = 9,  Fizica = 9, Chimie = 8,  Biologie = 10 },
    new Student { Nume = "Mihai Dumitrescu", Matematica = 6,  Informatica = 5,  Fizica = 7, Chimie = 6,  Biologie = 5  },
    new Student { Nume = "Ioana Stanescu",   Matematica = 8,  Informatica = 9,  Fizica = 8, Chimie = 7,  Biologie = 8  },
    new Student { Nume = "Cristian Popa",    Matematica = 4,  Informatica = 6,  Fizica = 5, Chimie = 4,  Biologie = 5  },
    new Student { Nume = "Andreea Dragomir", Matematica = 9,  Informatica = 8,  Fizica = 7, Chimie = 9,  Biologie = 8  },
};
```


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-10-printing/exercitiu.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
