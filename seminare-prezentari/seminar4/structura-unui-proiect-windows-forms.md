# Structura unui proiect Windows Forms

Aplicațiile scrise în laboratoarele anterioare rulează în consolă: citești date cu `Console.ReadLine()` și afișezi rezultate cu `Console.WriteLine()`. Interacțiunea este liniară, textuală și nu oferă utilizatorului nicio reprezentare vizuală a stării aplicației.

**Windows Forms** (prescurtat WinForms) este un framework pentru construirea de aplicații desktop cu interfață grafică în C# și .NET. Permite crearea de ferestre cu butoane, câmpuri de text, liste și alte elemente vizuale, plasate pe ecran și controlate prin mouse și tastatură.

Diferența față de consolă nu este doar vizuală. Modelul de execuție se schimbă fundamental. O aplicație consolă rulează instrucțiuni în ordine, de sus în jos, și se oprește când epuizează codul din `Main`. O aplicație WinForms pornește o **buclă de mesaje** care rulează continuu și asteaptă acțiuni ale utilizatorului. Când utilizatorul apasă un buton sau scrie în câmp, sistemul generează un eveniment, iar aplicația reacționează prin codul legat de acel eveniment. Programul nu avansează prin instrucțiuni secvențiale, ci prin reacții la acțiuni.

Această diferență explică de ce în WinForms nu mai scriem cod liniar în `Main`, ci scriem **event handlers** care se execută ca răspuns la interacțiunile utilizatorului.

### Crearea unui proiect WinForms

În Visual Studio, un proiect WinForms se creează selectând **Windows Forms App (.NET Framework)** din lista de tipuri de proiecte la crearea unui proiect nou. Este important să alegi varianta pentru `.NET Framework`, nu varianta `.NET` sau `.NET Core`, deoarece acestea au un sistem de designer ușor diferit față de cel prezentat în acest curs.

La crearea proiectului, Visual Studio generează automat o structură inițială funcțională: un formular gol, fișierele necesare și codul de lansare a aplicației. Proiectul poate fi rulat imediat, chiar dacă formularul este deocamdată gol.

### Fișierele generate automat

Un proiect WinForms nou conține trei fișiere principale, fiecare cu un rol precis.

#### Program.cs

Punctul de intrare al aplicației. Conține metoda `Main` care inițializează sistemul de afișare Windows și lansează formularul principal:

```csharp
static class Program
{
    [STAThread]
    static void Main()
    {
        Application.EnableVisualStyles();
        Application.SetCompatibleTextRenderingDefault(false);
        Application.Run(new Form1());
    }
}
```

`Application.Run(new Form1())` creează o instanță a formularului principal și pornește bucla de mesaje. Execuția rămâne în această linie până când utilizatorul închide fereastra. Nu modifici acest fișier în mod obișnuit, cu excepția cazului în care vrei să schimbi formularul de pornire sau să adaugi inițializare globală.

#### `Form1.cs`

Fișierul pe care îl editezi tu. Conține clasa `Form1`, constructorul ei și toți event handlerii pe care îi scrii. Aceasta este logica formularului:

```csharp
public partial class Form1 : Form
{
    private List<Contact> contacte = new List<Contact>();

    public Form1()
    {
        InitializeComponent();
    }

    private void btnAdauga_Click(object sender, EventArgs e)
    {
        // Logica butonului, scrisa de tine
    }
}
```

Tot codul care descrie comportamentul aplicației aparține acestui fișier.

#### `Form1.Designer.cs`

Fișierul generat automat de Designer-ul vizual din Visual Studio. Conține metoda `InitializeComponent()` care creează și configurează toate controalele adăugate prin interfața grafică:

```csharp
private void InitializeComponent()
{
    this.btnAdauga = new System.Windows.Forms.Button();
    this.txtNume = new System.Windows.Forms.TextBox();
    // ...
    this.btnAdauga.Text = "Adaugă contact";
    this.btnAdauga.Location = new System.Drawing.Point(12, 200);
    this.btnAdauga.Click += new System.EventHandler(this.btnAdauga_Click);
}
```

Fișierul este regenerat de fiecare dată când faci o modificare în Designer. Orice modificare manuală în el va fi suprascrisă sau va produce erori la redeschiderea formularului. Regula simplă este să nu editezi niciodată `Form1.Designer.cs` direct.

### Mecanismul `partial class`

Ambele fișiere, `Form1.cs` și `Form1.Designer.cs`, definesc aceeași clasă `Form1`, marcată cu cuvântul cheie `partial`:

```csharp
// Form1.cs — scris de tine
public partial class Form1 : Form { ... }

// Form1.Designer.cs — generat automat
partial class Form1 { ... }
```

Compilatorul unește cele două fișiere într-o singură clasă la compilare. Aceasta este tehnica prin care Visual Studio poate genera cod separat de codul tău fără conflicte. Din perspectiva compilatorului, există o singură clasă `Form1`; din perspectiva organizării fișierelor, codul tău și codul generat sunt separate și nu se amestecă.

### Designer-ul vizual și fereastra `Properties`

Dând dublu-click pe `Form1.cs` în Solution Explorer se deschide Designer-ul vizual: o reprezentare grafică a ferestrei tale, în care poți trage controale din Toolbox și le poți aranja vizual.

**Toolbox-ul** conține toate controalele disponibile, grupate în categorii. Categoria **Common Controls** conține cele mai folosite: `Button`, `Label`, `TextBox`, `ListBox`, `CheckBox`, `ComboBox`, `GroupBox`. Pentru a adăuga un control pe formular, îl tragi din Toolbox sau dai dublu-click pe el.

**Fereastra Properties** afișează toate proprietățile controlului selectat. Modificările făcute aici se reflectă automat în `Form1.Designer.cs`. Câmpul `(Name)` din Properties controlează numele cu care accesezi controlul în cod. Proprietatea `Text` controlează textul afișat. Sunt două lucruri diferite și este important să nu le confunzi.

Fereastra Properties are două moduri: lista de proprietăți și lista de evenimente. Comutarea între ele se face prin cele două iconițe din bara de sus a ferestrei Properties: una cu grilă (proprietăți) și una cu fulger (evenimente). Dând dublu-click pe un eveniment în lista de evenimente, Visual Studio generează automat handler-ul în `Form1.cs` și abonarea corespunzătoare în `Form1.Designer.cs`.


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-4-introducere-in-windows-forms/structura-unui-proiect-windows-forms.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
