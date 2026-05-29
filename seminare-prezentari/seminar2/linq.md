# LINQ

Orice program care lucrează cu colecții ajunge la aceleași operații: filtrare, sortare, transformare, agregare. Fără un mecanism dedicat, fiecare operație se scrie ca o buclă explicită cu variabile contor, condiții de filtrare și variabile de acumulare. Codul este verbose, ușor de greșit și dificil de citit dintr-o privire.

Considerați calculul sumei prețurilor finale ale tuturor biletelor valide:

```csharp
// Fara LINQ — verbose si predispus la erori
double total = 0;
foreach (Bilet b in bileteCumparate)
{
    if (b.EsteValid())
        total += b.CalculeazaPretFinal();
}
```

**LINQ** (Language Integrated Query) este un set de metode integrate în C# care exprimă aceste operații concis, citibil și componabil. Varianta cu LINQ:

```csharp
double total = bileteCumparate
    .Where(b => b.EsteValid())
    .Sum(b => b.CalculeazaPretFinal());
```

Ambele variante produc același rezultat. Diferența este că varianta LINQ exprimă **ce** vrei, nu **cum** să obții. Codul descrie intenția, nu mecanica.

Metodele LINQ operează pe orice colecție care implementează `IEnumerable<T>`, ceea ce include `List<T>`, array-uri și orice altă colecție standard. Pentru a le utiliza, adaugi `using System.Linq;` la începutul fișierului.

### Expresiile lambda în LINQ

Majoritatea metodelor LINQ primesc o funcție ca parametru: o instrucțiune aplicată fiecărui element al colecției. Această funcție se scrie inline cu sintaxa lambda: `element => expresie`. Mai multe detalii găsiți în cadrul Seminarului 3.

```csharp
b => b.CalculeazaPretFinal()
```

Se citește: *pentru fiecare `b`, evaluează `b.CalculeazaPretFinal()`*. `b` este parametrul. Numele este ales de tine și nu are nicio semnificație specială. Ce urmează după `=>` este expresia evaluată pe fiecare element.

Lambda-urile nu sunt o sintaxă specială inventată pentru LINQ. Sunt funcții anonime care pot fi transmise ca argumente oriunde este așteptat un tip `Func<T, TResult>` sau `Action<T>`. LINQ le utilizează extensiv, dar le-am întâlnit deja în capitolul despre delegați.

### Filtrare

#### `Where`

`Where` returnează elementele care satisfac o condiție. Primește un predicat (o funcție care returnează `bool` pentru fiecare element - detalii în Seminarul 3) și produce o colecție cu elementele pentru care predicatul este `true`:

```csharp
// Biletele cu pretul final peste 50 RON
List<Bilet> scumpe = bileteCumparate.Where(b => b.CalculeazaPretFinal() > 50).ToList();

// Biletele valide
List<Bilet> valide = bileteCumparate.Where(b => b.EsteValid()).ToList();

// Biletele unui anumit client — lambda captureaza variabila externa prin closure
string numeClient = "Ion Popescu";
List<Bilet> aleClientului = bileteCumparate
    .Where(b => b.Client.GetNumeComplet() == numeClient)
    .ToList();
```

#### `OfType<T>`

`OfType<T>` filtrează colecția după tipul real al obiectelor și returnează doar elementele de tipul `T` sau derivate din el. Spre deosebire de `Where`, nu verifică o condiție arbitrară, ci **tipul runtime** al fiecărui element:

```csharp
// Doar biletele de student
IEnumerable<BiletStudent> studenti = bileteCumparate.OfType<BiletStudent>();

// Doar biletele VIP
IEnumerable<BiletVIP> vip = bileteCumparate.OfType<BiletVIP>();
```

`OfType<T>` este deosebit de utilă în contextul colecțiilor heterogene (liste de tipul clasei de bază care conțin obiecte de tipuri derivate diferite) și se combină natural cu metodele generice din `CasaBilete`:

```csharp
public int GetNumarBiletePerTip<T>() where T : Bilet
{
    return bileteCumparate.OfType<T>().Count();
}
```

### Agregare

Metodele de agregare produc o singură valoare dintr-o colecție, aplicând o operație cumulativă pe toți membrii.

#### `Sum`

Calculează suma valorilor returnate de o expresie aplicată fiecărui element:

```csharp
double totalIncasari = bileteCumparate.Sum(b => b.CalculeazaPretFinal());
double totalReduceri = bileteCumparate.Sum(b => b.GetReducere());
```

#### `Count`

Numără elementele colecției, opțional cu un predicat de filtrare:

```csharp
int total      = bileteCumparate.Count();
int nrStudenti = bileteCumparate.Count(b => b is BiletStudent);
int nrValide   = bileteCumparate.Count(b => b.EsteValid());
```

#### `Min`, `Max` și `Average`

Returnează valoarea minimă, maximă sau media aritmetică a unei expresii:

```csharp
double pretMinim = bileteCumparate.Min(b => b.CalculeazaPretFinal());
double pretMaxim = bileteCumparate.Max(b => b.CalculeazaPretFinal());
double pretMediu = bileteCumparate.Average(b => b.CalculeazaPretFinal());
```

### Sortare

#### `OrderBy` și `OrderByDescending`

Sortează colecția crescător sau descrescător după o cheie specificată printr-un lambda:

```csharp
// Crescator dupa pretul final
List<Bilet> sortateAsc = bileteCumparate.OrderBy(b => b.CalculeazaPretFinal()).ToList();

// Descrescator dupa pretul final
List<Bilet> sortateDesc = bileteCumparate.OrderByDescending(b => b.CalculeazaPretFinal()).ToList();
```

#### `ThenBy`

Adaugă un criteriu secundar de sortare, aplicat când primul criteriu produce egalitate. Se înlănțuiește după `OrderBy` sau `OrderByDescending`:

```csharp
var sortate = bileteCumparate
    .OrderBy(b => b.NumeFilm)
    .ThenByDescending(b => b.CalculeazaPretFinal())
    .ToList();
```

### Extragerea de elemente individuale

#### `First` și `FirstOrDefault`

`First` returnează primul element al colecției sau primul care satisface un predicat. Aruncă excepție dacă colecția este goală sau dacă niciun element nu satisface condiția.

`FirstOrDefault` are același comportament, dar returnează `null` în loc să arunce excepție:

```csharp
// Cel mai scump bilet — sortam descrescator si luam primul
Bilet celMaiScump = bileteCumparate.OrderByDescending(b => b.CalculeazaPretFinal()).First();

// Primul bilet VIP, sau null daca nu exista niciun bilet VIP
Bilet primulVIP = bileteCumparate.FirstOrDefault(b => b is BiletVIP);
```

Alegerea între `First` și `FirstOrDefault` depinde de context: dacă absența elementului este o situație excepțională care trebuie semnalată ca eroare, `First` este potrivit. Dacă absența este o posibilitate normală, `FirstOrDefault` cu verificare `null` este mai adecvat.

#### `Single` și `SingleOrDefault`

Similare cu `First`, dar verifică în plus că există **exact un singur** element care satisface condiția. Aruncă excepție dacă există mai multe:

```csharp
// Biletul cu un anumit numar de loc — ar trebui sa fie unic in sala
Bilet biletLoc15 = bileteCumparate.SingleOrDefault(b => b.NumarLoc == 15);
```

### Transformare

#### `Select`

Transformă fiecare element dintr-o colecție într-o altă valoare, producând o nouă colecție cu rezultatele:

```csharp
// Colectia numelor filmelor
IEnumerable<string> filme = bileteCumparate.Select(b => b.NumeFilm);

// Colectia preturilor finale
IEnumerable<double> preturi = bileteCumparate.Select(b => b.CalculeazaPretFinal());
```

### Lazy loading și `ToList`

Un aspect important al LINQ este că metodele de filtrare și transformare returnează un `IEnumerable<T>` care este **lazy loaded**. Calculul efectiv are loc abia când parcurgi rezultatul, nu la momentul apelului metodei LINQ.

Aceasta înseamnă că înlănțuirea mai multor metode LINQ nu produce rezultate intermediare. Se construiește o descriere a transformărilor, care este executată o singură dată când datele sunt efectiv necesare.

Dacă ai nevoie de o colecție **materializată imediat** (de exemplu, pentru a o stoca sau a o parcurge de mai multe ori), apelezi `ToList()` sau `ToArray()` la finalul lanțului:

```csharp
List<Bilet> bileteValide = bileteCumparate.Where(b => b.EsteValid()).ToList();
```

Fără `ToList()`, variabila `bileteValide` ar conține o interogare care se re-execută de fiecare dată când este parcursă, nu o listă calculată o singură dată.

### Înlănțuirea metodelor

Metodele LINQ pot fi **înlănțuite**: rezultatul uneia devine intrarea celeilalte. Aceasta permite exprimarea unor interogări complexe într-un format citibil, fiecare linie adăugând un pas de transformare:

```csharp
List<Bilet> topTrei = bileteCumparate
    .Where(b => b.EsteValid())
    .OrderByDescending(b => b.CalculeazaPretFinal())
    .Take(3)
    .ToList();
```

Se citește de sus în jos: din toate biletele, păstrează-le pe cele valide, sortează-le descrescător după preț, ia primele trei, materializează ca listă. Fiecare metodă exprimă o transformare precisă, iar înlănțuirea lor produce o interogare compozită clară.


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-2-programarea-orientata-obiect-in-c/linq.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
