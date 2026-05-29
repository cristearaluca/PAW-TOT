# Câmpuri și proprietăți

Cel mai simplu mod de a stoca date într-o clasă este un câmp public: o variabilă declarată direct în clasă, vizibilă și modificabilă din orice alt loc din program. Această abordare funcționează tehnic, dar nu oferă niciun control asupra valorilor acceptate.

Considerați varianta cu câmpuri publice pentru clasa `Client`:

```csharp
class Client
{
    public string Telefon;
    public string Email;
}
```

Orice cod din program poate scrie orice valoare, fără nicio verificare:

```csharp
Client c = new Client();
c.Telefon = "abc";        // invalid, dar compilatorul nu se opune
c.Telefon = "";           // la fel
c.Telefon = null;         // la fel
c.Email = "nu_e_email";   // la fel
```

Obiectul ajunge într-o stare invalidă fără să știe. Bug-ul va apărea mai târziu, departe de locul unde s-a produs, și va fi dificil de localizat. Nu există niciun punct central unde să fie impusă o regulă. Validarea ar trebui repetată manual la fiecare atribuire, în tot codul care lucrează cu obiecte `Client`.

Soluția este să nu permitem accesul direct la memorie din exterior. **Proprietățile** fac exact asta.

### Ce este o proprietate?

O proprietate arată din exterior ca un câmp simplu. Se citește și se atribuie cu aceeași sintaxă, dar în interior conține **logică executabilă**. Are două componente: `get`, care controlează citirea, și `set`, care controlează scrierea.

```csharp
class Client
{
    private string _telefon;  // campul privat — invizibil din exterior

    public string Telefon    // proprietatea publica — vizibila din exterior
    {
        get { return _telefon; }
        set
        {
            if (value == null || value.Length != 10)
                throw new ArgumentException("Telefonul trebuie sa aiba exact 10 caractere.");
            _telefon = value;
        }
    }
}
```

Din exterior, utilizarea arată identic cu un câmp:

```csharp
client.Telefon = "0721000001";   // apeleaza set
string t = client.Telefon;       // apeleaza get
```

Diferența esențială este că atribuirea trece acum prin `set`, care poate refuza valoarea înainte ca aceasta să ajungă în memorie. Dacă transmitem `"abc"`, se aruncă o excepție imediat, în locul exact în care s-a produs problema.

### Câmpul privat și proprietatea publică

Convenția fundamentală în programarea orientată pe obiecte este: **câmpurile sunt private, proprietățile sunt publice**.

Câmpul privat (`_telefon`) este detaliul de implementare. Nimeni din afara clasei nu știe că există și nu îl poate accesa direct. Proprietatea (`Telefon`) este contractul cu exteriorul: alți programatori utilizează proprietatea și nu au nevoie să știe cum este stocată valoarea intern.

Această separare aduce un avantaj practic imediat. Dacă decidem ulterior să stocăm telefonul într-un format diferit (de exemplu, fără spații sau puncte), modificăm doar implementarea internă a câmpului privat și a proprietății. Codul care folosește `client.Telefon` nu se schimbă și nici nu are nevoie să fie recompilat.

### Cuvântul cheie `value`

În blocul `set`, cuvântul cheie `value` reprezintă valoarea pe care apelantul încearcă să o atribuie. Nu este un parametru declarat explicit, ci este furnizat automat de compilator ori de câte ori este compilat un accesor `set`.

```csharp
set
{
    // value contine ce s-a scris in dreapta semnului =
    // client.Telefon = "0721000001"  =>  value este "0721000001"

    if (value == null || value.Length != 10)
        throw new ArgumentException("Telefonul trebuie sa aiba exact 10 caractere.");

    _telefon = value;  // abia acum scriem in memorie, dupa validare
}
```

Structura tipică a unui `set` cu validare este mereu aceeași: verifici `value`, arunci excepție dacă nu respectă regulile, atribui câmpului privat dacă este valid. Valoarea invalidă nu ajunge niciodată în memorie.

### Proprietatea `Email` - validare cu condiție compusă

Clasa `Client` din exercițiu impune ca emailul să conțină caracterele `@` și `.`. Proprietatea exprimă această regulă o singură dată, în mod centralizat:

```csharp
private string _email;

public string Email
{
    get { return _email; }
    set
    {
        if (value == null || !value.Contains("@") || !value.Contains("."))
            throw new ArgumentException("Emailul trebuie sa contina @ si .");
        _email = value;
    }
}
```

Oriunde în program se atribuie `client.Email = "..."`, validarea rulează automat. Nu trebuie să ne amintim să o apelăm manual. Nu există nicio cale de a ajunge la un obiect `Client` cu un email invalid, atâta timp cât câmpul este privat și accesul se face exclusiv prin proprietate.

### Proprietăți auto-implementate

Când nu există nevoie de validare, scrierea unui câmp privat împreună cu o proprietate completă devine verboasă și repetitivă. C# oferă o sintaxă prescurtată numită **proprietate auto-implementată**:

```csharp
public string Nume { get; set; }
```

Compilatorul generează automat câmpul privat și metodele de acces. Efectul este identic cu:

```csharp
private string _nume;
public string Nume
{
    get { return _nume; }
    set { _nume = value; }
}
```

În clasa `Client`, proprietățile fără restricții speciale (`Nume` și `Prenume`) pot fi declarate în forma auto-implementată. Proprietățile cu validare (`Email` și `Telefon`) necesită forma completă, deoarece `set` trebuie să conțină logică explicită.

Alegerea între cele două forme nu este o chestiune de preferință estetică. Auto-implementată înseamnă „accept orice valoare", forma completă înseamnă „am reguli pe care le impun". Utilizarea greșită a formei auto-implementate acolo unde ar trebui validare este una dintre sursele frecvente de stare invalidă în obiecte.

### Proprietăți read-only

Uneori dorim ca o valoare să fie setată o singură dată (în general în constructor) și să nu mai poată fi modificată ulterior din exterior. Există două mecanisme pentru aceasta.

**`private set`**: proprietatea poate fi citită din orice loc, dar poate fi scrisă doar din interiorul clasei.

```csharp
public string NumarLegitimatie { get; private set; }
```

**Numai `get`**: valoarea poate fi setată exclusiv prin constructor și nu mai poate fi modificată ulterior în niciun fel.

```csharp
public string CodUnic { get; }
```

Aceste forme de proprietate read-only sunt o formă de **încapsulare mai strictă**: nu doar ascundem câmpul, ci interzicem explicit modificarea din exterior după inițializare. Obiectele imuabile, ale căror valori nu se schimbă după creare, sunt mult mai ușor de raționat și de depanat, deoarece starea lor este predictibilă pe toată durata de viață.

### Principiul încapsulării

Tot ce am descris în acest capitol urmează un singur principiu fundamental: **obiectul își controlează propria stare**.

Un obiect nu este o structură de date pasivă în care oricine poate scrie orice. Este o entitate cu reguli proprii, care decide ce valori acceptă, cum le stochează și în ce condiții le modifică. Codul exterior comunică prin proprietăți și metode, nu prin acces direct la câmpurile interne.

Beneficiul concret este că un obiect valid rămâne valid pe toată durata existenței sale. Dacă toate validările sunt concentrate în proprietăți și în constructor, nu există nicio cale ca un obiect să ajungă într-o stare invalidă prin utilizare normală, ci fiecare atribuire trece prin logica de verificare.

Comparând cele două abordări:

```csharp
// Fara encapsulare: obiectul poate fi in orice stare dupa creare
Client c = new Client();
// c.Telefon este null, c.Email este null — obiectul exista dar este inutilizabil

// Cu encapsulare si constructor: obiectul este garantat valid dupa creare
Client c = new Client("Ion", "Popescu", "ion@gmail.com", "0721000001");
// daca parametrii sunt invalizi, constructorul arunca exceptie si obiectul nu este creat
// daca constructorul se termina, obiectul este garantat in stare valida
```

Constructorii — mecanismul prin care impunem că un obiect este valid de la momentul creării — sunt subiectul capitolului următor.


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-2-programarea-orientata-obiect-in-c/campuri-si-proprietati.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
