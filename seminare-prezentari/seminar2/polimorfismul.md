# Polimorfismul

Polimorfismul este consecința directă a perechii `virtual`/`override`, prezentată în capitolul anterior. Odată ce mai multe clase definesc versiuni diferite ale aceleiași metode, apare o întrebare fundamentală: când apelezi metoda pe un obiect al cărui tip exact nu îl cunoști la momentul scrierii codului, ce versiune rulează?

Răspunsul este întotdeauna: **versiunea corespunzătoare tipului real al obiectului din memorie**, nu tipului variabilei prin care îl accesezi.

Aceasta este definiția practică a polimorfismului. Nu este un concept abstract, ci un comportament concret și verificabil al runtime-ului C#.

### Un exemplu pas cu pas

Creăm patru obiecte de tipuri diferite și le plasăm într-o listă de tip `Bilet`:

```csharp
List<Bilet> bilete = new List<Bilet>
{
    new Bilet(..., pretBaza: 30),
    new BiletStudent(..., pretBaza: 30),
    new BiletSenior(..., pretBaza: 30, varstaClient: 65),
    new BiletVIP(..., pretBaza: 60, includePopcorn: true, includeBautura: false)
};
```

Lista este de tip `List<Bilet>`. Compilatorul acceptă toate cele patru obiecte deoarece fiecare este fie un `Bilet`, fie o clasă derivată din el. Toate respectă relația „este un" prezentată la moștenire. Din perspectiva listei, toate sunt bilete.

Acum parcurgem lista și apelăm `GetReducere()` pe fiecare element:

```csharp
foreach (Bilet b in bilete)
{
    Console.WriteLine($"Reducere: {b.GetReducere()} RON");
}
```

Variabila `b` este de tip `Bilet` la fiecare iterație. Dar obiectul din memorie la care `b` pointează este de tipul real cu care a fost creat. C# caută implementarea lui `GetReducere()` pe acel tip real:

```
Reducere: 0 RON    ← Bilet.GetReducere() — returnează 0
Reducere: 6 RON    ← BiletStudent.GetReducere() — returnează 30 * 0.20
Reducere: 9 RON    ← BiletSenior.GetReducere() — returnează 30 * 0.30
Reducere: 0 RON    ← BiletVIP.GetReducere() — returnează 0 (are GetExtras() în schimb)
```

Același apel `b.GetReducere()`, scris o singură dată în cod, produce patru rezultate diferite în funcție de tipul real al fiecărui obiect. Acesta este polimorfismul.

### De ce este utilă această proprietate?

Fără polimorfism, codul care lucrează cu colecții mixte ar trebui să verifice manual tipul fiecărui obiect și să aplice logica corespunzătoare:

```csharp
// Varianta fara polimorfism — de evitat
foreach (Bilet b in bilete)
{
    double reducere;
    if (b is BiletStudent)
        reducere = b.PretBaza * 0.20;
    else if (b is BiletSenior)
        reducere = b.PretBaza * 0.30;
    else
        reducere = 0;

    Console.WriteLine($"Reducere: {reducere} RON");
}
```

Această abordare are o problemă gravă de mentenabilitate: de fiecare dată când adăugăm un tip nou de bilet, trebuie să găsim și să modificăm **toate** locurile din cod care fac astfel de verificări manuale. Dacă uităm unul singur, obținem un bug silențios. Codul compilează fără erori, dar produce rezultate greșite.

Cu polimorfism, fiecare clasă își cunoaște propria logică și o exprimă prin `override`. Codul care parcurge colecția nu știe și nu trebuie să știe câte tipuri există:

```csharp
// Varianta cu polimorfism
foreach (Bilet b in bilete)
{
    Console.WriteLine($"Reducere: {b.GetReducere()} RON");
}
```

Dacă adăugăm mâine `BiletCopil` cu reducere de 50%, nu modificăm nimic în `foreach`. Pur și simplu clasa nouă implementează `override double GetReducere()` și totul funcționează automat, fără nicio intervenție în codul care consumă colecția.

### Polimorfismul în `CasaBilete`

`CasaBilete` menține o colecție privată de tip `List<Bilet>` și calculează totalul încasărilor fără să cunoască tipurile concrete ale biletelor stocate:

```csharp
class CasaBilete
{
    private List<Bilet> _bileteCumparate = new List<Bilet>();

    public double GetIncasariTotale()
    {
        double total = 0;
        foreach (Bilet b in _bileteCumparate)
            total += b.CalculeazaPretFinal();  // apeleaza implementarea corecta pentru fiecare tip
        return total;
    }

    public double GetReduceriAcordate()
    {
        double total = 0;
        foreach (Bilet b in _bileteCumparate)
            total += b.GetReducere();  // 0, 6, 9, 0, etc. in functie de tipul real
        return total;
    }
}
```

`CasaBilete` a fost scrisă o singură dată și funcționează corect pentru orice combinație de tipuri de bilete, inclusiv tipuri care nu existau când a fost scrisă. Aceasta este puterea reală a polimorfismului: codul care consumă obiecte nu depinde de tipurile concrete ale acelor obiecte.

### Polimorfism prin interfețe

Polimorfismul funcționează și prin interfețe, nu doar prin moștenire. Dacă mai multe clase implementează aceeași interfață, pot fi tratate uniform prin tipul interfeței, indiferent dacă au vreo relație de moștenire între ele:

```csharp
List<IPretCalculabil> bilete = new List<IPretCalculabil>
{
    new Bilet(...),
    new BiletStudent(...),
    new BiletVIP(...)
};

double total = 0;
foreach (IPretCalculabil b in bilete)
    total += b.CalculeazaPretFinal();
```

`IPretCalculabil` nu știe nimic despre reduceri, extras-uri sau tipuri concrete. Știe doar că obiectele care o implementează pot calcula un preț final. Atât îi este necesar. Fiecare obiect din listă aplică propria logică, conform propriei implementări a interfeței.

### `is` și `as` - când tipul concret contează

Polimorfismul rezolvă elegant cazurile în care vrem să tratăm toate obiectele uniform. Există totuși situații în care tipul concret al obiectului chiar contează (de exemplu, când vrem să afișăm informații specifice fiecărui tip, informații care nu există pe clasa de bază).

**Pattern matching cu `is`** verifică tipul și, dacă se potrivește, extrage o referință tipizată într-un singur pas:

```csharp
foreach (Bilet b in bilete)
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

Dacă `b` este de tipul `BiletStudent`, condiția este `true` și variabila `bs` este populată automat cu referința tipizată, gata de utilizat în blocul `if`. Dacă nu, condiția este `false` și blocul este ignorat.

Ordinea ramurilor contează: tipurile mai specifice trebuie verificate înaintea celor mai generale. Dacă `BiletStudentVIP` moștenește din `BiletVIP`, ramura `b is BiletVIP` ar prinde și obiectele de tip `BiletStudentVIP`. Tipul mai specific trebuie verificat primul.

Regulă practică: **polimorfism prin `virtual`/`override`** pentru logică care se aplică uniform tuturor obiectelor, **`is`/`as`** pentru cazurile în care comportamentul diferă structural și informațiile necesare nu există pe clasa de bază.


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-2-programarea-orientata-obiect-in-c/polimorfismul.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
