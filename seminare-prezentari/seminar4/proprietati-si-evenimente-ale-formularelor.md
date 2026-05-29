# Proprietăți și evenimente ale formularelor

`Form` este clasa de bază pentru orice fereastră în WinForms. Formularul principal al aplicației, ferestrele de dialog și ferestrele secundare sunt toate instanțe ale unor clase care moștenesc din `Form`.

Clasa `Form1` generată automat la crearea proiectului moștenește `Form` și poate fi personalizată în două moduri complementare: prin proprietăți vizuale setate în Designer, și prin cod scris în `Form1.cs`. Ambele moduri produc același efect la runtime. Diferența este doar unde se află setarea în fișierele proiectului.

### Proprietățile esențiale ale unui formular

Proprietățile unui formular se pot seta fie din fereastra Properties în Designer, fie direct în cod în constructorul `Form1()`, după apelul `InitializeComponent()`.

#### `Text`

Textul afișat în bara de titlu a ferestrei:

```csharp
this.Text = "Registru Contacte";
```

#### `Size`

Dimensiunea ferestrei în pixeli, exprimată ca un obiect `Size` cu lățimea și înălțimea:

```csharp
this.Size = new Size(600, 500);
```

Alternativ, proprietatea `ClientSize` setează dimensiunea zonei de conținut, excluzând bara de titlu și borduri. Dacă vrei să controlezi exact spațiul disponibil pentru controale, `ClientSize` este mai precis.

#### `StartPosition`

Determină unde apare fereastra pe ecran la lansarea aplicației:

```csharp
this.StartPosition = FormStartPosition.CenterScreen;
```

Valoarea `CenterScreen` centrează fereastra pe ecran și este cea mai folosită în practică. Alte valori posibile sunt `WindowsDefaultLocation` (sistemul decide), `Manual` (poziție setată explicit prin proprietatea `Location`) și `CenterParent` (centrată față de fereastra părinte, utilă pentru dialoguri).

#### `FormBorderStyle`

Controlează aspectul bordurii ferestrei și ce butoane de control sunt disponibile în bara de titlu:

```csharp
this.FormBorderStyle = FormBorderStyle.FixedSingle;
```

Valorile frecvent folosite sunt `Sizable` (implicit, fereastra poate fi redimensionată de utilizator), `FixedSingle` (bordură fixă, nu poate fi redimensionată, toate butoanele de control sunt prezente), `FixedDialog` (aspect de dialog, fără butonul de minimize) și `None` (fără bordură, fără bare de titlu).

#### `MaximizeBox` și `MinimizeBox`

Afișează sau ascunde butoanele de maximize și minimize din bara de titlu. Setarea unuia nu o afectează pe cealaltă:

```csharp
this.MaximizeBox = false;
this.MinimizeBox = true;
```

Dacă `FormBorderStyle` este `FixedSingle`, setarea `MaximizeBox = false` este utilă pentru a comunica utilizatorului că fereastra nu poate și nu trebuie redimensionată.

#### `BackColor` și `Font`

Culoarea de fundal a formularului și fontul implicit aplicat tuturor controalelor care nu au un font propriu setat:

```csharp
this.BackColor = Color.WhiteSmoke;
this.Font = new Font("Segoe UI", 9f);
```

Setarea `Font` pe formular propagă fontul la toate controalele copil care nu au font propriu. Aceasta este o modalitate eficientă de a asigura un aspect uniform fără a seta fontul individual pe fiecare control.

### Setarea proprietăților în Designer față de cod

Ambele variante produc același rezultat la runtime. Alegerea depinde de natura proprietății.

Proprietățile statice, care nu depind de date din aplicație, se setează convenabil din Designer: `Text`, `Size`, `FormBorderStyle`, `StartPosition`. Modificările sunt imediat vizibile în reprezentarea grafică a formularului.

Proprietățile care depind de date sau de logica aplicației aparțin codului:

```csharp
public Form1()
{
    InitializeComponent();

    // Valoare calculata -- nu poate fi setata in Designer
    this.Text = "Registru Contacte -- " + DateTime.Now.Year;
}
```

### Ciclul de viață al unui formular

Un formular parcurge mai multe etape de la creare la închidere. Fiecare etapă corespunde unui eveniment la care te poți abona pentru a executa cod la momentul potrivit.

#### `Load`

Se declanșează după ce formularul este creat și toate controalele sunt inițializate prin `InitializeComponent()`, dar înainte ca fereastra să fie afișată pe ecran. Acesta este locul potrivit pentru inițializări: popularea unui `ListBox`, setarea valorilor implicite, încărcarea datelor din orice sursă.

```csharp
private void Form1_Load(object sender, EventArgs e)
{
    lblStatus.Text = "Aplicatie pornita. Niciun contact incarcat.";
    txtNume.Focus();
}
```

Este important să înțelegi că, înainte de `Load`, controalele există în memorie (au fost create de `InitializeComponent()`), dar fereastra nu este vizibilă. Poți accesa și modifica controalele fără probleme.

#### `Shown`

Se declanșează după ce formularul este afișat pentru prima dată pe ecran. Diferența față de `Load` este că la momentul `Shown` fereastra este deja vizibilă. Util pentru a muta focusul pe un anumit control sau pentru animații de intrare.

```csharp
private void Form1_Shown(object sender, EventArgs e)
{
    txtNume.Focus();
}
```

#### `FormClosing`

Se declanșează când utilizatorul încearcă să închidă fereastra, înainte ca aceasta să se închidă efectiv. Permite anularea închiderii sau afișarea unui dialog de confirmare. Evenimentul primește un argument `FormClosingEventArgs` cu proprietatea `Cancel`: dacă o setezi pe `true`, fereastra nu se va închide.

```csharp
private void Form1_FormClosing(object sender, FormClosingEventArgs e)
{
    if (contacte.Count > 0)
    {
        DialogResult rezultat = MessageBox.Show(
            "Ai " + contacte.Count + " contacte nesalvate. Inchizi aplicatia?",
            "Confirmare",
            MessageBoxButtons.YesNo,
            MessageBoxIcon.Question);

        if (rezultat == DialogResult.No)
            e.Cancel = true;
    }
}
```

#### `FormClosed`

Se declanșează după ce fereastra s-a închis efectiv. Util pentru eliberarea resurselor sau salvarea stării aplicației înainte de ieșire. La momentul `FormClosed`, fereastra nu mai este vizibilă, dar obiectul `Form` mai există în memorie.

### Abonarea la evenimentele unui formular

Cel mai simplu mod de a abona un eveniment al formularului este prin Designer: dai click pe o zonă goală a formularului (pentru a selecta formularul, nu un control), deschizi tab-ul de evenimente din fereastra Properties (iconița cu fulger) și dai dublu-click pe evenimentul dorit.

Visual Studio generează automat handler-ul în `Form1.cs` și linia de abonare în `Form1.Designer.cs`:

```csharp
// Generat in Form1.Designer.cs:
this.Load += new System.EventHandler(this.Form1_Load);
this.FormClosing += new System.Windows.Forms.FormClosingEventHandler(this.Form1_FormClosing);

// Generat in Form1.cs (tu completezi corpul):
private void Form1_Load(object sender, EventArgs e)
{
    // codul tau
}
```

O alternativă la tab-ul de evenimente este dublu-click direct pe suprafața goală a formularului în Designer: aceasta generează automat handler-ul pentru evenimentul `Load`, care este cel mai frecvent folosit.


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-4-introducere-in-windows-forms/proprietati-si-evenimente-ale-formularelor.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
