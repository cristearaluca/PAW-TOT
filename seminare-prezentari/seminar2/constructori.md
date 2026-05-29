# Constructori

Când creăm un obiect cu `new` fără să definim un constructor, C# îl inițializează cu valori implicite: `null` pentru tipuri referință, `0` pentru numere, `false` pentru `bool`.

```csharp
Client c = new Client();
Console.WriteLine(c.Nume);    // null
Console.WriteLine(c.Email);   // null
Console.WriteLine(c.Telefon); // null
```

Obiectul există în memorie, dar se află într-o stare inutilizabilă. Dacă transmitem acest obiect la o metodă care presupune că are un email valid, sau dacă apelăm `c.GetNumeComplet()` care concatenează `Prenume` și `Nume`, obținem fie un rezultat incorect, fie o excepție `NullReferenceException`. În cel mai nefericit caz, eroarea va apărea în cu totul altă parte a programului decât locul unde s-a produs problema.

Sursa deficienței este că am separat crearea obiectului de inițializarea lui. Obiectul a putut fi creat fără ca datele necesare să fie furnizate. Constructorul rezolvă exact această problemă.

### Ce este constructorul și cum funcționează?

Constructorul este o **metodă specială apelată automat de operatorul `new`** în momentul creării obiectului. Are același nume ca și clasa și nu are tip de retur, nici măcar `void`.

```csharp
class Client
{
    public string Nume { get; set; }
    public string Prenume { get; set; }
    public string Email { get; set; }
    public string Telefon { get; set; }

    public Client(string nume, string prenume, string email, string telefon)
    {
        Nume = nume;
        Prenume = prenume;
        Email = email;
        Telefon = telefon;
    }
}
```

Odată definit acest constructor, `new Client()` fără parametri nu mai compilează. Singurul mod de a crea un client este să furnizezi toate datele de la început:

```csharp
Client c = new Client("Ion", "Popescu", "ion@gmail.com", "0721000001");
```

Efectul este că **obiectul nu poate exista fără date**. Nu este o restricție arbitrară, este o garanție: oricine primește un obiect `Client` știe că are toate câmpurile completate. Nu trebuie să verifice dacă `Nume` este `null` înainte de a-l folosi. Nu trebuie să se întrebe dacă obiectul a fost corect inițializat.

### Constructorul și validarea din proprietăți

O decizie importantă de implementare este că, în constructor, valorile sunt atribuite prin **proprietăți**, nu direct în câmpuri. Aceasta înseamnă că validarea din blocurile `set` rulează automat în timpul construcției, fără niciun cod suplimentar.

```csharp
public string Telefon
{
    get { return _telefon; }
    set
    {
        if (value == null || value.Length != 10)
            throw new ArgumentException("Telefonul trebuie sa aiba exact 10 caractere!");
        _telefon = value;
    }
}
```

Dacă apelăm:

```csharp
Client c = new Client("Ion", "Popescu", "ion@gmail.com", "123");
```

...constructorul încearcă să atribuie `"abc"` proprietății `Telefon`. Proprietatea aruncă `ArgumentException`. Constructorul nu finalizează. Obiectul nu ajunge în memorie.

Consecința practică este că nu poți obține niciodată un obiect `Client` cu un număr de telefon invalid.  Validarea nu trebuie apelată manual de fiecare dată. Ea este garantată structural, prin modul în care clasa este construită.

Acesta este motivul pentru care validarea se pune în proprietăți și se atribuie prin proprietăți în constructor, nu după construcție: un obiect invalid nu ar trebui să poată exista.

### Constructori multipli

O clasă poate defini **mai mulți constructori**, cu liste de parametri diferite. Mecanismul se numește supraîncărcare și permite crearea obiectelor în moduri diferite, în funcție de ce date sunt disponibile la momentul creării.

```csharp
class Bilet
{
    public string NumeFilm { get; set; }
    public int NumarSala { get; set; }
    public double PretBaza { get; set; }
    public Client Client { get; set; }

    // Constructor complet — cu client
    public Bilet(string numeFilm, int numarSala, double pretBaza, Client client)
    {
        NumeFilm = numeFilm;
        NumarSala = numarSala;
        PretBaza = pretBaza;
        Client = client;
    }

    // Constructor partial — clientul poate fi asociat ulterior
    public Bilet(string numeFilm, int numarSala, double pretBaza)
    {
        NumeFilm = numeFilm;
        NumarSala = numarSala;
        PretBaza = pretBaza;
    }
}
```

Compilatorul alege constructorul potrivit în funcție de numărul și tipul argumentelor transmise la `new`. Dacă niciun constructor nu se potrivește, codul nu compilează — eroarea este detectată devreme.

#### Delegarea între constructori cu `: this(...)`

Când mai mulți constructori conțin cod comun, duplicarea poate fi evitată prin delegare: un constructor îl apelează pe altul cu `: this(...)`.

```csharp
// Constructorul cu 4 parametri il apeleaza pe cel cu 3
public Bilet(string numeFilm, int numarSala, double pretBaza, Client client)
    : this(numeFilm, numarSala, pretBaza)
{
    // corpul poate fi gol sau poate adauga logica suplimentara
    Client = client;
}
```

Constructorul delegat rulează primul, urmat de corpul constructorului care a făcut delegarea. Aceasta asigură că validările și inițializările comune au loc o singură dată, indiferent de ce constructor este apelat.

### Constructorul în clasele derivate -  `: base(...)`

Când o clasă moștenește altă clasă, constructorul clasei derivate trebuie să asigure inițializarea și a **părții moștenite** din obiect. Aceasta se realizează cu `: base(...)`, care apelează explicit constructorul clasei de bază.

```csharp
class BiletStudent : Bilet
{
    public string NumarLegitimatie { get; set; }
    public string Facultate { get; set; }

    public BiletStudent(string numeFilm, int numarSala, double pretBaza, Client client,
                        string numarLegitimatie, string facultate)
        : base(numeFilm, numarSala, pretBaza, client)  // initializeaza partea Bilet
    {
        NumarLegitimatie = numarLegitimatie;
        Facultate = facultate;
    }
}
```

Ordinea de execuție este întotdeauna **de sus în jos în ierarhie**: mai întâi rulează constructorul clasei de bază (`Bilet`), abia apoi continuă constructorul clasei derivate (`BiletStudent`). Nu poți accesa sau inițializa membrii clasei de bază din constructorul derivat înainte ca constructorul de bază să fi rulat. Compilatorul asigură această ordine.

Dacă uiți `: base(...)` și clasa de bază nu are constructor fără parametri, codul nu compilează. Compilatorul nu are cum să știe cum să inițializeze partea moștenită.

Importanța practică este că validările din clasa de bază rulează și în constructorii claselor derivate. Dacă `Bilet` validează că prețul de bază este pozitiv, atunci un `BiletStudent` cu preț negativ nu poate fi creat. Validarea este moștenită implicit prin lanțul de constructori.

### Constructorul implicit

Dacă nu definești niciun constructor, compilatorul C# generează automat un **constructor implicit** fără parametri care nu face nimic. Acesta este motivul pentru care `new Client()` funcționează pe o clasă fără constructor definit explicit.

De îndată ce definești cel puțin un constructor cu parametri, constructorul implicit generat automat **dispare**. Dacă mai ai nevoie și de un constructor fără parametri, trebuie să îl declari explicit:

```csharp
class Client
{
    public string Nume { get; set; }

    // Un constructor cu parametri a fost definit
    public Client(string nume)
    {
        Nume = nume;
    }

    // Fara aceasta linie, new Client() nu ar compila
    public Client() { }
}
```

Acesta este un comportament care surprinde frecvent: adaugi un constructor cu parametri la o clasă existentă și brusc tot codul care folosea `new Client()` fără parametri nu mai compilează. Motivul este tocmai că constructorul implicit a dispărut odată cu adăugarea celui explicit.


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-2-programarea-orientata-obiect-in-c/constructori.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
