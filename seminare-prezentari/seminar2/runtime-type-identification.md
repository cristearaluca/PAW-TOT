# Runtime Type Identification

Polimorfismul prin `virtual`/`override` rezolvă elegant cazurile în care vrem să tratăm toate obiectele dintr-o colecție uniform: apelăm aceeași metodă pe fiecare element și fiecare își execută propria versiune. Codul care parcurge colecția nu știe și nu trebuie să știe ce tipuri concrete există.

Există, totuși, situații în care tipul concret chiar contează. Dacă vrem să afișăm un raport detaliat al tuturor biletelor vândute și să includem informații specifice fiecărui tip (numărul de legitimație pentru biletele student, vârsta clientului pentru biletele senior, serviciile suplimentare pentru biletele VIP), avem nevoie să accesăm proprietăți care nu există pe clasa de bază `Bilet`. Polimorfismul nu ajută în acest caz: `GetReducere()` poate fi apelat uniform, dar `NumarLegitimatie` nu există decât pe `BiletStudent`.

C# oferă doi operatori pentru a lucra corect cu tipuri concrete într-un context polimorfic: `is` și `as`.

### Operatorul `is` - verificarea tipului

În forma sa de bază, `is` verifică dacă un obiect este de un anumit tip și returnează `true` sau `false`:

```csharp
Bilet b = new BiletStudent(...);

if (b is BiletStudent)
    Console.WriteLine("Este un bilet de student.");
```

Această formă verifică tipul, dar nu oferă acces la proprietățile specifice. Pentru a le accesa, am fi nevoiți să facem un cast explicit separat.

#### Pattern matching cu `is`

C# 7 a introdus o formă mai compactă numită **pattern matching**, care combină verificarea tipului și extragerea unei referințe tipizate într-un singur pas:

```csharp
if (b is BiletStudent bs)
{
    // bs este de tip BiletStudent, disponibila in tot blocul if
    Console.WriteLine($"Student: {bs.NumarLegitimatie}, Facultate: {bs.Facultate}");
}
```

Dacă `b` este de tipul `BiletStudent` sau derivat din el, condiția este `true` și variabila `bs` este populată automat cu o referință tipizată. Dacă nu se potrivește, condiția este `false` și `bs` nu este accesibilă. Aceasta elimină castul explicit și reduce codul la o singură linie de verificare.

### Tratarea diferențiată a tipurilor dintr-o ierarhie

Pattern matching strălucește când vrem să tratăm fiecare tip dintr-o ierarhie diferit, de exemplu în contextul afișării unui raport:

```csharp
foreach (Bilet b in bileteCumparate)
{
    if (b is BiletStudent bs)
        Console.WriteLine($"{b.NumeFilm} | Student: {bs.Facultate}, legitimatie {bs.NumarLegitimatie}");
    else if (b is BiletSenior bsen)
        Console.WriteLine($"{b.NumeFilm} | Senior: {bsen.VarstaClient} ani");
    else if (b is BiletVIP bv)
        Console.WriteLine($"{b.NumeFilm} | VIP, extras: {bv.GetExtras()} RON");
    else
        Console.WriteLine($"{b.NumeFilm} | Bilet standard");
}
```

**Ordinea ramurilor contează.** Dacă în ierarhie există `BiletStudentVIP` care moștenește din `BiletVIP`, ramura `b is BiletVIP` ar prinde și obiectele de tip `BiletStudentVIP`. Tipurile mai specifice trebuie verificate înaintea celor mai generale:

```csharp
if (b is BiletStudentVIP bsvip)    // mai specific — primul
    Console.WriteLine("Student VIP");
else if (b is BiletVIP bv)         // mai general — al doilea
    Console.WriteLine("VIP standard");
```

### Operatorul `as`

Operatorul `as` încearcă să convertească un obiect la un tip și returnează referința tipizată dacă reușește sau `null` dacă nu:

```csharp
BiletStudent bs = b as BiletStudent;

if (bs != null)
    Console.WriteLine(bs.NumarLegitimatie);
```

Spre deosebire de castul explicit `(BiletStudent)b`, operatorul `as` **nu aruncă excepție** dacă tipul nu se potrivește. Acesta returnează `null` în tăcere și lasă verificarea în seama programatorului. Dacă uiți verificarea `null`, riscul de `NullReferenceException` se mută în alt loc.

### Compararea celor trei mecanisme

Există trei modalități de a converti un obiect la un tip mai specific, cu comportamente diferite la eșec:

```csharp
Bilet b = new BiletSenior(...);

// Cast explicit — arunca InvalidCastException daca tipul nu se potriveste
BiletStudent bs1 = (BiletStudent)b;    // exceptie la runtime

// as — returneaza null daca tipul nu se potriveste
BiletStudent bs2 = b as BiletStudent;  // bs2 este null, fara exceptie

// is cu pattern matching — verifica si extrage intr-un singur pas
if (b is BiletStudent bs3)             // false, bs3 nu e accesibila
    Console.WriteLine(bs3.NumarLegitimatie);
```

În practică, **pattern matching cu `is`** este varianta preferată pentru verificări condiționale care combină siguranța cu concizia. **`as` cu verificare `null`** este util când ai nevoie de referință în afara unui bloc `if`. **Castul explicit** se folosește exclusiv când ești sigur de tip și dorești ca programul să eșueze zgomotos dacă te înșeli.

### `switch` cu pattern matching

O alternativă la lanțul de `if`/`else if` este `switch` cu pattern matching, disponibil din C# 7. Produce același comportament, dar poate fi mai lizibil când există mai multe tipuri de tratat:

```csharp
switch (b)
{
    case BiletStudent bs:
        Console.WriteLine($"Student: {bs.NumarLegitimatie}");
        break;
    case BiletSenior bsen:
        Console.WriteLine($"Senior: {bsen.VarstaClient} ani");
        break;
    case BiletVIP bv:
        Console.WriteLine($"VIP: {bv.GetExtras()} RON extras");
        break;
    default:
        Console.WriteLine("Bilet standard");
        break;
}
```

Și în `switch`, ordinea ramurilor contează pentru tipuri din aceeași ierarhie, iar ramurile mai specifice trebuie plasate înaintea celor mai generale.

### Când folosim RTTI față de polimorfism?

Apariția frecventă a construcțiilor `is`/`as` în cod este uneori un semnal de proiectare: dacă tot ce dorești de la fiecare tip este un comportament specific, probabil că acel comportament ar trebui exprimat ca metodă `virtual`/`override` pe clasa de bază.

Cu toate acestea, există cazuri legitime în care RTTI este instrumentul corect. Afișarea unui raport detaliat cu informații specifice fiecărui tip este un exemplu. Datele afișate sunt structural diferite și nu pot fi exprimate printr-o singură metodă polimorfică. Regula practică: **polimorfism prin `virtual`/`override` pentru logică uniformă**, **`is`/`as` pentru cazurile în care comportamentul diferă structural și datele accesate nu există pe clasa de bază**.


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-2-programarea-orientata-obiect-in-c/runtime-type-identification.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
