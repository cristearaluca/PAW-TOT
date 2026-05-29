# Moștenirea

Sistemul de bilete al unui cinematograf gestionează mai multe tipuri de bilete: standard, de student, de senior și VIP. Toate au în comun un set de proprietăți: filmul, sala, data de expirare, clientul, locul și prețul de bază. Ceea ce le diferențiază sunt câteva proprietăți suplimentare și modul de calcul al prețului final.

Fără un mecanism dedicat, am avea două opțiuni, ambele cu deficiențe grave.

Prima opțiune este să scriem `BiletStudent`, `BiletSenior` și `BiletVIP` ca **clase complet independente**, fiecare cu toate proprietățile comune repetate. Dacă decidem să adăugăm un câmp nou (de exemplu, numărul rândului) trebuie modificate toate trei clasele. Dacă există un bug în logica comună, trebuie corectat în trei locuri. Codul comun, repetat, este o sursă constantă de inconsistențe.

A doua opțiune este o **singură clasă `Bilet`** cu toate proprietățile posibile, unele rămânând `null` sau nefolosite pentru tipurile care nu le necesită. Codul devine greu de înțeles, iar obiectele au o structură ambiguă. Nu poți spune dintr-o privire ce tip de bilet reprezintă un anumit obiect.

**Moștenirea** oferă a treia cale: scriem o dată ceea ce este comun și extindem doar ce este diferit.

### Relația „este un"

Moștenirea exprimă o relație de tip **„este un"** între clase. `BiletStudent` este un `Bilet`. `BiletSenior` este un `Bilet`. Această relație nu este o simplă convenție de cod, ci reflectă realitatea domeniului. Un bilet de student rămâne un bilet, cu toate proprietățile și regulile unui bilet standard, la care se adaugă câteva specifice.

Testul practic este direct: dacă propoziția „X este un Y" are sens în domeniu, atunci X poate moșteni Y. Dacă nu are sens, moștenirea nu este potrivită:

```
BiletStudent este un Bilet    → moștenire justificată
BiletSenior este un Bilet     → moștenire justificată
CasaBilete este un Bilet      → nu are sens în domeniu, nu moștenim
```

### Sintaxa de bază

Moștenirea se declară cu `:` urmat de numele clasei de bază:

```csharp
class BiletStudent : Bilet
{
    public string NumarLegitimatie { get; set; }
    public string Facultate { get; set; }

    public BiletStudent(string numeFilm, int numarSala, TipFilm tipFilm,
                        DateTime expiraLa, Client client, int numarLoc, double pretBaza,
                        string numarLegitimatie, string facultate)
        : base(numeFilm, numarSala, tipFilm, expiraLa, client, numarLoc, pretBaza)
    {
        NumarLegitimatie = numarLegitimatie;
        Facultate = facultate;
    }
}
```

Constructorul lui `BiletStudent` primește toți parametrii necesari, îi transmite pe cei comuni către `Bilet` prin `: base(...)` și îi stochează pe cei proprii. Rezultatul este că un obiect `BiletStudent` conține toate câmpurile lui `Bilet` plus `NumarLegitimatie` și `Facultate`.

### Ce se moștenește?

Clasa derivată primește automat tot ce este `public` sau `protected` din clasa de bază: proprietăți, metode, câmpuri. Membrii `private` nu se moștenesc. Aceștia sunt strict interni clasei care i-a definit și nu sunt vizibili nicăieri în afara ei.

Modificatorul `protected` merită o atenție specială: un membru `protected` este invizibil din exterior (la fel ca `private`), dar este vizibil în clasele derivate. Este mecanismul prin care o clasă de bază oferă acces controlat la anumite detalii interne claselor care o extind, fără să le expună și codului extern.

```csharp
class Bilet
{
    private double pretBaza;                   // invizibil chiar si in clasele derivate
    public string NumeFilm { get; set; }       // vizibil oriunde
    protected int numarLoc;                    // vizibil in derivate, invizibil din exterior

    public double CalculeazaPretFinal() { ... } // vizibil oriunde
}
```

Un obiect `BiletStudent` poate folosi `NumeFilm` și `CalculeazaPretFinal()` direct, de parcă ar fi definite în el. Câmpul privat `pretBaza` nu este accesibil direct, dar poate fi citit prin proprietatea publică corespunzătoare.

```csharp
BiletStudent bs = new BiletStudent(...);
Console.WriteLine(bs.NumeFilm);               // moștenita din Bilet
Console.WriteLine(bs.CalculeazaPretFinal());  // moștenita din Bilet
Console.WriteLine(bs.NumarLegitimatie);       // proprie lui BiletStudent
```

### Ierarhia din exercițiu

Sistemul de bilete are trei clase derivate directe din `Bilet`:

```
Bilet
├── BiletStudent   (adaugă NumarLegitimatie, Facultate; reducere 20%)
├── BiletSenior    (adaugă VarstaClient; reducere 30%)
└── BiletVIP       (adaugă IncludePopcorn, IncludeBautura; extras în loc de reducere)
```

Fiecare extinde `Bilet` în direcția sa proprie. Codul comun (validarea locului, calculul prețului final, verificarea datei de expirare) stă o singură dată în `Bilet`.

### Moștenire pe mai multe niveluri

Moștenirea nu se limitează la un singur nivel. O clasă derivată poate fi la rândul ei extinsă, creând o ierarhie cu mai mulți niveluri. Dacă am dori un bilet care combină avantajele VIP cu o reducere de student, am crea `BiletStudentVIP` ca derivată din `BiletVIP`:

```csharp
class BiletStudentVIP : BiletVIP
{
    public string NumarLegitimatie { get; set; }

    public BiletStudentVIP(string numeFilm, int numarSala, TipFilm tipFilm,
                           DateTime expiraLa, Client client, int numarLoc, double pretBaza,
                           bool includePopcorn, bool includeBautura, string numarLegitimatie)
        : base(numeFilm, numarSala, tipFilm, expiraLa, client, numarLoc, pretBaza,
               includePopcorn, includeBautura)
    {
        NumarLegitimatie = numarLegitimatie;
    }
}
```

Ierarhia extinsă devine:

```
Bilet
├── BiletStudent
├── BiletSenior
└── BiletVIP
    └── BiletStudentVIP
```

Un obiect `BiletStudentVIP` conține câmpurile tuturor celor trei clase. Moștenirea se propagă complet de sus în jos.

### O singură clasă de bază

În C#, o clasă poate moșteni **o singură altă clasă**. Moștenirea multiplă (din două clase de bază în același timp) nu este permisă:

```csharp
class BiletStudentVIP : BiletVIP, BiletStudent  // eroare de compilare
```

Această restricție există pentru a evita ambiguitățile care apar când două clase de bază definesc aceeași metodă. Dacă ai nevoie de comportament din surse multiple, soluția sunt **interfețele.**

### Modificatorul `sealed`

Uneori dorim să interzicem extinderea unei clase. Modificatorul `sealed` face ca o clasă să nu poată fi moștenită:

```csharp
sealed class BiletSenior : Bilet
{
    // nicio altă clasă nu poate moșteni BiletSenior
}
```

`sealed` se folosește când logica unei clase este suficient de specifică și completă încât extinderea ei nu ar avea sens conceptual sau ar putea produce comportamente incorecte. În exercițiul nostru nu avem nevoie de `sealed`, dar merită cunoscut ca mecanism de protecție a ierarhiei.


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-2-programarea-orientata-obiect-in-c/mostenirea.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
