# Rezumat

### Structura unui proiect WinForms

Un proiect WinForms este organizat în trei fișiere cu roluri distincte și fixe.

`Program.cs` este punctul de intrare al aplicației. Conține metoda `Main` care pornește sistemul de afișare Windows și lansează formularul principal prin `Application.Run(new Form1())`. Nu se modifică în mod obișnuit.

`Form1.cs` este fișierul pe care îl editezi tu. Conține logica formularului: lista de date din memorie, event handlerii și metodele care implementează funcționalitățile aplicației.

`Form1.Designer.cs` este generat automat de Designer-ul vizual și conține metoda `InitializeComponent()` care creează și configurează toate controalele. Nu se editează manual, orice modificare directă în el va fi suprascrisă sau va produce erori.

Cele două fișiere `Form1.cs` și `Form1.Designer.cs` definesc aceeași clasă prin mecanismul `partial`. Compilatorul le unește la compilare.

| Fisier              | Rol                                         | Editat de           |
| ------------------- | ------------------------------------------- | ------------------- |
| `Program.cs`        | Punctul de intrare, lanseaza formularul     | Rar, de tine        |
| `Form1.cs`          | Logica formularului, event handlers, metode | Tu                  |
| `Form1.Designer.cs` | Codul generat pentru controale              | Automat de Designer |

### Formularul (`Form`) - proprietăți si evenimente

Proprietățile statice se setează în Designer; proprietățile care depind de date se setează în cod, după `InitializeComponent()`.

| Proprietate                   | Ce controleaza                                                   |
| ----------------------------- | ---------------------------------------------------------------- |
| `Text`                        | Textul din bara de titlu                                         |
| `Size`                        | Dimensiunea ferestrei in pixeli                                  |
| `StartPosition`               | Unde apare fereastra la lansare (`CenterScreen` cel mai folosit) |
| `FormBorderStyle`             | Tipul bordurii: `Sizable`, `FixedSingle`, `FixedDialog`, `None`  |
| `MaximizeBox` / `MinimizeBox` | Afiseaza sau ascunde butoanele de maximize si minimize           |

| Eveniment     | Cand se declanseaza                                               |
| ------------- | ----------------------------------------------------------------- |
| `Load`        | Inainte de afisare, dupa initializare - pentru setup initial      |
| `Shown`       | La prima afisare pe ecran - pentru `Focus()` si animatii          |
| `FormClosing` | La tentativa de inchidere - `e.Cancel = true` anuleaza inchiderea |
| `FormClosed`  | Dupa inchiderea efectivă - pentru cleanup                         |

### Controale de bază

Convenția de denumire cu prefix scurt se aplică imediat la adăugarea fiecărui control:

```
btn  ->  Button       txt  ->  TextBox      lbl  ->  Label
lst  ->  ListBox      chk  ->  CheckBox     grp  ->  GroupBox
cmb  ->  ComboBox
```

| Control   | Proprietate cheie                      | Eveniment principal |
| --------- | -------------------------------------- | ------------------- |
| `Label`   | `Text`, `AutoSize`, `ForeColor`        | nu are              |
| `TextBox` | `Text`, `PlaceholderText`, `MaxLength` | `TextChanged`       |
| `Button`  | `Text`, `Enabled`                      | `Click`             |

`sender` în event handlers este controlul care a declanșat evenimentul, transmis ca `object`. Dacă ai nevoie de proprietăți specifice, faci cast: `Button btn = sender as Button`.

### ListBox

`ListBox`-ul și lista din memorie sunt două lucruri separate. Lista din memorie (`List<Contact>`) este sursa de adevăr. `ListBox`-ul este oglinda ei vizuală. `RefreshLista()` este metoda care le ține sincronizate.

```csharp
private void RefreshLista()
{
    lstContacte.Items.Clear();
    foreach (Contact c in contacte)
        lstContacte.Items.Add(c);
}
```

`ListBox`-ul afișează rezultatul `ToString()` al fiecărui obiect. Suprascrierea `ToString()` pe clasa `Contact` controlează ce text apare în listă.

| Proprietate sau Metoda | Ce face                                                  |
| ---------------------- | -------------------------------------------------------- |
| `Items.Add(obiect)`    | Adauga un element                                        |
| `Items.Clear()`        | Goleste lista vizuala                                    |
| `SelectedItem`         | Elementul selectat ca `object`; null daca nimic selectat |
| `SelectedIndex`        | Indexul elementului selectat; -1 daca nimic selectat     |
| `SelectionMode`        | `One` (implicit), `MultiSimple`, `MultiExtended`         |

| Eveniment              | Cand se declanseaza            |
| ---------------------- | ------------------------------ |
| `SelectedIndexChanged` | La orice schimbare a selectiei |

La accesul la elementul selectat, foloseste `SelectedItem as Contact`, nu `SelectedIndex` pentru a indexa lista din memorie. Aceasta evita inconsistente in prezenta filtrarii.

### GroupBox, CheckBox, MessageBox

| Control      | Proprietate cheie                           | Eveniment principal       |
| ------------ | ------------------------------------------- | ------------------------- |
| `GroupBox`   | `Text`, `Enabled` (dezactiveaza tot grupul) | nu are                    |
| `CheckBox`   | `Text`, `Checked`                           | `CheckedChanged`          |
| `MessageBox` | apel static `Show(...)`                     | returneaza `DialogResult` |

`MessageBox.Show` în forma completa cu captarea raspunsului:

```csharp
DialogResult rezultat = MessageBox.Show(
    "Mesajul afisat utilizatorului",
    "Titlul ferestrei",
    MessageBoxButtons.YesNo,
    MessageBoxIcon.Question
);
if (rezultat == DialogResult.Yes) { ... }
```

| `MessageBoxButtons` | Butoane afisate   |
| ------------------- | ----------------- |
| `OK`                | Un singur buton   |
| `OKCancel`          | OK si Cancel      |
| `YesNo`             | Yes si No         |
| `YesNoCancel`       | Yes, No si Cancel |

### Bune practici

| Regula                                           | Motivul                                           |
| ------------------------------------------------ | ------------------------------------------------- |
| Nu scrie logica in event handlers                | Codul devine greu de citit si de testat           |
| Redenumeste controalele imediat                  | `button1` nu comunica nimic dupa o saptamana      |
| Proprietati statice in Designer, dinamice in cod | Separare clara intre fix si variabil              |
| Validare cu `MessageBox`, nu exceptii            | Exceptiile in handlers afecteaza utilizatorul     |
| `.Trim()` la citirea textului                    | Elimina spatii accidentale din TextBox-uri        |
| `Focus()` dupa erori de validare                 | Utilizatorul stie exact unde sa corecteze         |
| Actualizeaza UI dupa date, nu inainte            | Consistenta intre memorie si ce vede utilizatorul |


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-4-introducere-in-windows-forms/rezumat.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
