# Enumerații

Să presupunem că vrem să stocăm tipul unui film pe un bilet. Prima variantă care vine în minte este un `string`:

```csharp
class Bilet
{
    public string TipFilm { get; set; }
}
```

Funcționează tehnic, dar nu oferă nicio garanție asupra valorilor acceptate. Orice cod din program poate scrie orice:

```csharp
bilet.TipFilm = "Comedie";
bilet.TipFilm = "comedie";    // acelasi lucru? sau tip diferit?
bilet.TipFilm = "COMEDIE";    // si asta?
bilet.TipFilm = "komedye";    // greseala de tastare — compilatorul nu observa
bilet.TipFilm = "SF";         // tip care nu exista in sistemul nostru
```

Toate aceste atribuiri compilează fără erori. Niciuna nu este semnalată ca problemă. Bug-urile apar mai târziu, când codul încearcă să compare sau să proceseze valoarea și găsește ceva neașteptat — o literă mare în loc de mică, o greșeală de tastare, un tip care nu ar trebui să existe.

Rădăcina problemei este că `string` poate reprezenta orice, iar noi vrem să reprezentăm o mulțime finită și bine definită de valori. Avem nevoie de un tip care să exprime exact această constrângere.

### Ce este o enumerație?

O **enumerație** este un tip care poate lua **doar una dintre valorile declarate explicit**. Se definește cu cuvântul cheie `enum`:

```csharp
enum TipFilm
{
    Comedie,
    Actiune,
    Drama,
    Istoric
}
```

`TipFilm` este acum un tip de sine stătător, la fel ca `int` sau `string`. O variabilă de acest tip poate conține exclusiv una dintre cele patru valori declarate. Orice altceva este respins de compilator:

```csharp
TipFilm tip = TipFilm.Drama;     // corect
TipFilm tip = TipFilm.SF;        // eroare de compilare: SF nu exista in enumeratie
TipFilm tip = "Drama";           // eroare de compilare: string nu este TipFilm
```

Greșelile de tipul `"komedye"` devin imposibile. Nu există niciun string de comparat, nicio capitalizare de verificat, nicio valoare surpriză de tratat. Compilatorul verifică la fiecare atribuire că valoarea aparține mulțimii definite.

### Utilizarea enumerației în clasă

Proprietatea din clasa `Bilet` devine de tipul `TipFilm` în loc de `string`:

```csharp
class Bilet
{
    public string NumeFilm { get; set; }
    public TipFilm TipFilm { get; set; }
    // ...
}
```

Atribuirea și citirea funcționează cu aceeași sintaxă ca orice altă proprietate:

```csharp
Bilet b = new Bilet();
b.TipFilm = TipFilm.Comedie;

if (b.TipFilm == TipFilm.Comedie)
{
    Console.WriteLine("Film de comedie.");
}
```

Codul este explicit și fără ambiguitate. Nu mai trebuie să ne amintim formatul exact al stringului, nu există inconsistențe de capitalizare și nu există posibilitatea ca un tip inexistent să ajungă în sistem.

### Enumerații în `switch`

Un caz de utilizare frecvent și natural pentru enumerații este instrucțiunea `switch`, unde dorim să tratăm diferit fiecare valoare posibilă. Enumerațiile se potrivesc perfect pentru acest pattern deoarece mulțimea cazurilor posibile este finită și cunoscută la compilare:

```csharp
switch (bilet.TipFilm)
{
    case TipFilm.Comedie:
        Console.WriteLine("Pregateste-te sa razi.");
        break;
    case TipFilm.Actiune:
        Console.WriteLine("Efecte speciale garantate.");
        break;
    case TipFilm.Drama:
        Console.WriteLine("Ai batiste la tine?");
        break;
    case TipFilm.Istoric:
        Console.WriteLine("Bazat pe fapte reale.");
        break;
}
```

Dacă adăugăm ulterior o nouă valoare în enumerație (de exemplu `SF`), compilatorul însuși nu avertizează automat că avem un `case` lipsă în `switch`, dar IDE-ul o va semnala și este mult mai ușor să găsim toate locurile afectate decât dacă am fi folosit string-uri. Oricum suntem cu mult mai în control față de varianta bazată pe string.

### Reprezentarea numerică internă

În memorie, o enumerație este stocată ca un număr întreg. Prin convenție, primul element are valoarea `0`, al doilea `1` și așa mai departe, dacă nu specificăm altfel:

```csharp
Console.WriteLine((int)TipFilm.Comedie);  // 0
Console.WriteLine((int)TipFilm.Actiune);  // 1
Console.WriteLine((int)TipFilm.Drama);    // 2
Console.WriteLine((int)TipFilm.Istoric);  // 3
```

Putem atribui valori numerice explicite fiecărui element, ceea ce este util când enumerația trebuie să corespundă unor coduri dintr-o bază de date sau dintr-un protocol extern:

```csharp
enum TipFilm
{
    Comedie = 1,
    Actiune = 2,
    Drama   = 3,
    Istoric = 4
}
```

Conversia din număr în enumerație este posibilă, dar necesită un cast explicit. Aceasta poate eșua dacă numărul nu corespunde niciunei valori definite:

```csharp
TipFilm tip = (TipFilm)2;  // TipFilm.Drama (daca valorile sunt cele de mai sus)
```

În exercițiul nostru nu avem nevoie de valori numerice explicite. Le menționăm deoarece sunt o sursă frecventă de confuzie atunci când enumerațiile sunt serializate sau stocate în baze de date. Valoarea numerică stocată trebuie să corespundă valorii din cod, inclusiv după refactorizări care reordonează elementele.

### Conversia la `string` și din `string`

Când afișăm o valoare de tip enumerație sau o convertim la `string`, C# returnează automat **numele valorii**:

```csharp
TipFilm tip = TipFilm.Drama;
Console.WriteLine(tip);            // Drama
Console.WriteLine(tip.ToString()); // Drama
```

Conversia inversă (din `string` în enumerație) se face cu `Enum.Parse`:

```csharp
TipFilm tip = (TipFilm)Enum.Parse(typeof(TipFilm), "Drama");
Console.WriteLine(tip); // Drama
```

`Enum.Parse` aruncă excepție dacă string-ul nu corespunde niciunei valori definite. Varianta mai sigură, care returnează `false` în loc să arunce excepție, este `Enum.TryParse`:

```csharp
if (Enum.TryParse("Drama", out TipFilm tip))
{
    Console.WriteLine($"Tip valid: {tip}");
}
else
{
    Console.WriteLine("Valoare invalida.");
}
```

`Enum.TryParse` este de preferat ori de câte ori valoarea string provine dintr-o sursă externă (intrare de la utilizator, fișier, rețea) unde nu putem garanta că valoarea este validă.


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-2-programarea-orientata-obiect-in-c/enumeratii.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
