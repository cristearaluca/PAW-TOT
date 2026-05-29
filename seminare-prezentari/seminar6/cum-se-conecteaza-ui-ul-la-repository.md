# Cum se conectează UI-ul la Repository

### Prima variantă: fiecare formular își creează propriul repository

Cel mai intuitiv mod în care un formular accesează un repository este să și-l instanțieze singur. Atât `Form1` cât și `FormCarte` declară un câmp privat de tip `CarteRepository` și îl creează cu `new`:

```csharp
// Form1.cs
public partial class Form1 : Form
{
    private CarteRepository _repository = new CarteRepository();
    // ...
}

// FormCarte.cs
public partial class FormCarte : Form
{
    private CarteRepository _repository = new CarteRepository();
    // ...
}
```

La prima vedere pare rezonabil — fiecare formular are acces la repository, constructorul lui `FormCarte` este simplu, nu există dependențe vizibile între formulare.

### De ce această abordare funcționează cu `FakeDatabase`

Spre deosebire de laboratorul anterior unde lista de angajați trăia în `Form1`, acum datele trăiesc în `FakeDatabase.Carti` - o listă **statică**. Aceasta înseamnă că există o singură copie a ei în memorie, indiferent de câte instanțe de `CarteRepository` sunt create.

```csharp
// Form1 are instanta A de repository
CarteRepository repoA = new CarteRepository();

// FormCarte are instanta B de repository
CarteRepository repoB = new CarteRepository();

// Ambele acceseaza aceeasi lista statica
repoA.Add(new Carte { Titlu = "Clean Code" });

repoA.GetAll().Count;  // 1
repoB.GetAll().Count;  // 1 — vede aceeasi carte, pentru ca FakeDatabase.Carti e statica
```

Ceea ce scrie `FormCarte` prin instanța sa de repository ajunge în `FakeDatabase.Carti` (aceeași listă pe care o citește `Form1` prin instanța sa). Aplicația funcționează corect în aparență.

### Problema mascată: nu repository-ul rezolvă, ci `static`

Deși aplicația funcționează, există o problemă subtilă de înțelegere: **corectitudinea funcționării nu vine din arhitectura repository-ului, ci din caracterul static al `FakeDatabase`**. Repository-ul în sine nu face nimic pentru a garanta că toate instanțele lucrează cu aceleași date — pur și simplu se întâmplă că datele sunt statice.

Dacă mâine înlocuim `FakeDatabase` cu o bază de date reală, instanțele de repository vor fi instanțe separate de conexiuni, cu cache-uri separate, și problema reapare. Arhitectura care funcționează acum s-ar strica fără nicio modificare aparentă în structura codului.

Această situație ilustrează o problemă mai generală: **un cod care funcționează nu este neapărat un cod corect**. Funcționează din motive care s-ar putea schimba.

### Problema reală: instanțe multiple fără sursă de date statică

Să vedem ce s-ar întâmpla dacă datele nu ar fi statice. De exemplu, dacă `CarteRepository` ar deține propria colecție:

```csharp
// Daca repository-ul ar detine datele (gresit arhitectural, dar util pentru ilustrare)
class CarteRepository
{
    private List<Carte> carti = new List<Carte>();
    public List<Carte> GetAll() { return new List<Carte>(carti); }
    public void Add(Carte c) { carti.Add(c); }
}
```

```
Memoria aplicatiei in acest caz:

Form1
  └── repository  ──►  [instanta A]
                         carti: [ ]  ← goala

FormCarte
  └── repository  ──►  [instanta B]
                         carti: [ Carte("Clean Code") ]  ← cartea adaugata
```

`Form1` citește din instanța A. `FormCarte` a scris în instanța B. Datele nu circulă. Aplicația nu aruncă nicio eroare. Din perspectiva ei, totul funcționează corect. Bug-ul este silențios, ceea ce îl face și mai periculos.

Ori de câte ori scrii `new CarteRepository()`, C# alocă memorie pentru un obiect nou și complet independent. Nu există nicio legătură automată între două instanțe ale aceleiași clase, chiar dacă tipul lor este identic.

### Ce avem nevoie: o singură instanță partajată

Soluția robustă, care funcționează indiferent de natura sursei de date, este ca toate formularele să lucreze cu **același obiect** `CarteRepository`. Există mai multe moduri de a realiza asta.

### Transmiterea prin constructor

Cea mai simplă și mai transparentă soluție: `Form1` creează o singură instanță și o transmite lui `FormCarte` prin constructor. Ambele formulare lucrează cu același obiect, indiferent de ce se află în spatele repository-ului.

```csharp
// Form1 — creeaza instanta o singura data
public partial class Form1 : Form
{
    private CarteRepository repository = new CarteRepository();

    private void btnAdauga_Click(object sender, EventArgs e)
    {
        // Transmitem instanta existenta, nu cream una noua
        using (var f = new FormCarte(repository, null))
        {
            if (f.ShowDialog() == DialogResult.OK)
                RefreshLista();
        }
    }
}

// FormCarte — primeste instanta, nu o creeaza
public partial class FormCarte : Form
{
    private CarteRepository repository;

    public FormCarte(CarteRepository repository, int? id)
    {
        InitializeComponent();
        this.repository = repository;
        this.id = id;
    }
}
```

Acum ambele variabile `repository` din `Form1` și `FormCarte` pointează către **același obiect** din memorie. Nu mai depindem de statici pentru a garanta că datele sunt partajate. Arhitectura funcționează corect indiferent de cum este implementat repository-ul intern.

Avantajul major al acestei abordări este că dependența este **explicită și vizibilă**. Citind constructorul lui `FormCarte`, este imediat clar că are nevoie de un repository și că îl primește din exterior. Nu există magie ascunsă.

### Singleton: o singură instanță la nivel de aplicație

Există situații în care transmiterea prin constructor devine greoaie - de exemplu, când repository-ul trebuie să ajungă la un formular deschis printr-un lanț lung de ferestre. Pattern-ul **Singleton** rezolvă asta garantând că o clasă are o singură instanță în toată aplicația și oferind un punct global de acces la ea.

Repository-ul devine responsabil pentru propria instanțiere:

```csharp
class CarteRepository
{
    // Instanta unica, stocata static — apartine clasei, nu unui obiect
    private static CarteRepository instanta;

    // Constructor privat — nimeni din exterior nu poate face "new CarteRepository()"
    private CarteRepository() { }

    // Punct global de acces — creeaza instanta la primul apel, o returneaza pe urmatoarele
    public static CarteRepository GetInstanta()
    {
        if (instanta == null)
            instanta = new CarteRepository();
        return instanta;
    }

    // Metodele CRUD raman identice
    public List<Carte> GetAll() { return new List<Carte>(FakeDatabase.Carti); }
    // ...
}
```

Orice formular poate accesa repository-ul fără să primească o referință de undeva:

```csharp
// Form1 — fara parametru
private CarteRepository repository = CarteRepository.GetInstanta();

// FormCarte — acelasi apel, acelasi obiect returnat
private CarteRepository repository = CarteRepository.GetInstanta();
```

Singleton elimină necesitatea transmiterii prin constructor, dar introduce dezavantaje pe care e bine să le cunoști: este dificil de testat în izolare, creează o dependență globală implicită care nu se vede în semnăturile constructorilor, și poate crea probleme în aplicații cu mai multe fire de execuție dacă nu este implementat cu grijă.

### Dependency Injection: instanța gestionată din exterior

**Dependency Injection** (DI) este o abordare mai generală în care obiectele nu își creează dependențele — le primesc din exterior. Transmiterea prin constructor pe care am văzut-o mai sus este cel mai simplu exemplu de DI manual.

Principiul este că clasa declară de ce are nevoie, iar altcineva — codul apelant sau un container DI — se ocupă de creare și transmitere:

```csharp
// FormCarte declara ca are nevoie de un CarteRepository
// Nu stie si nu ii pasa cum a fost creat sau ce implementare are
public FormCarte(CarteRepository repository, int? id)
{
    this.repository = repository;
}
```

În aplicații mai mari, un container DI gestionează automat crearea și transmiterea dependențelor. Înregistrezi o singură dată că `CarteRepository` trebuie să fie un singleton, iar containerul îl injectează automat oriunde este nevoie — fără să transmiți manual prin fiecare constructor. Acesta este modelul standard în ASP.NET Core, de exemplu.

### Unde suntem și ce urmează

Exercițiul din acest laborator funcționează corect datorită `FakeDatabase` static, chiar și cu instanțe multiple de repository. Dar această corectitudine este fragilă și depinde de o caracteristică a sursei de date, nu de arhitectura repository-ului.

Soluția robustă și recomandată pentru aplicații WinForms de dimensiunea acestor laboratoare este transmiterea prin constructor. Singleton și DI complet sunt subiecte mai avansate, pentru cursuri de arhitectură software sau pentru momentul în care aplicațiile devin suficient de mari încât să justifice complexitatea adițională.

<table><thead><tr><th>Abordare</th><th width="250">Cum funcționează</th><th>Avantaje</th><th>Dezavantaje</th></tr></thead><tbody><tr><td><strong>Instanțiere locală</strong></td><td><code>new Repository()</code> în fiecare formular</td><td>Simplu de scris</td><td>Corect doar cu surse statice; fragil altfel</td></tr><tr><td><strong>Transmitere prin constructor</strong></td><td><code>Form1</code> creează, transmite prin <code>new FormCarte(repo, id)</code></td><td>Explicit, ușor de urmărit, robust</td><td>Lanț lung la multe niveluri</td></tr><tr><td><strong>Singleton</strong></td><td><code>Repository.GetInstanta()</code> returnează mereu același obiect</td><td>Accesibil global, fără transmitere</td><td>Greu de testat, dependență implicită</td></tr><tr><td><strong>Dependency Injection</strong></td><td>Container gestionează crearea și livrarea</td><td>Flexibil, testabil, scalabil</td><td>Complexitate suplimentară</td></tr></tbody></table>


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-6-separarea-responsabilitatilor-listview/cum-se-conecteaza-ui-ul-la-repository.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
