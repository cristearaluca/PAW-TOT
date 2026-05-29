# Interfețe

Moștenirea oferă reutilizare de cod: clasa derivată primește automat proprietățile și metodele clasei de bază. Dar uneori nu avem nevoie de reutilizare de cod, avem nevoie de o **garanție**: certitudinea că un anumit obiect, indiferent de tipul său concret, pune la dispoziție o operație specifică.

O interfață exprimă exact această garanție. Este un **contract**: declară ce metode trebuie să existe, fără să spună nimic despre cum sunt implementate. Orice clasă care semnează contractul (adică îl implementează) se angajează să furnizeze toate metodele declarate. Compilatorul verifică respectarea acestui angajament.

O analogie concretă: o priză electrică este un contract. Nu îi pasă ce aparat îi conectezi, atâta timp cât aparatul respectă formatul prizei. Aparatul nu trebuie să știe cum funcționează rețeaua din spatele prizei. Ambele părți respectă contractul și interacțiunea funcționează. Interfața joacă rolul prizei; clasa care o implementează este aparatul.

### Sintaxa și declararea

O interfață se declară cu `interface` și conține **exclusiv semnăturile metodelor**, fără corp, fără câmpuri, fără modificatori de acces pe membri:

```csharp
interface IPretCalculabil
{
    double CalculeazaPretFinal();
    double GetReducere();
}

interface IValidabil
{
    bool EsteValid();
}
```

Nu există `public`, `private` sau `virtual` pe metodele dintr-o interfață. Prin convenție, toate sunt publice și trebuie implementate. Numele interfețelor începe prin convenție cu litera `I`, semnalând explicit că este vorba de un contract, nu de o clasă.

### Implementarea unei interfețe

O clasă declară că respectă contractul cu același `:` folosit la moștenire, urmat de numele interfeței. Poate implementa oricâte interfețe, separate prin virgulă:

```csharp
class Bilet : IPretCalculabil, IValidabil
{
    public double PretBaza { get; set; }
    public DateTime ExpiraLa { get; set; }

    public double CalculeazaPretFinal()
    {
        return PretBaza - GetReducere();
    }

    public double GetReducere()
    {
        return 0;
    }

    public bool EsteValid()
    {
        return ExpiraLa > DateTime.Now;
    }
}
```

Dacă o clasă declară că implementează o interfață dar nu definește toate metodele din ea, codul **nu compilează**. Compilatorul verifică că fiecare metodă din contract are o implementare. Nu există nicio cale de a „uita" să implementezi o metodă din interfață.

### Interfață față de moștenire

Moștenirea și interfețele utilizează același simbol `:`, dar rezolvă probleme diferite.

**Moștenirea** exprimă relația „este un" și aduce **cod reutilizabil** din clasa de bază. `BiletStudent` moștenește `Bilet` și primește automat toate câmpurile și metodele acestuia.

**Interfața** exprimă relația „poate face" și impune un **contract** fără a contribui cu nicio linie de cod. `Bilet` implementează `IPretCalculabil` și garantează că știe să calculeze un preț, dar interfața nu furnizează nicio implementare.

Diferența practică cea mai importantă: în C# poți moșteni **o singură clasă**, dar poți implementa **oricâte interfețe**:

```csharp
// O singura clasa de baza
class BiletStudent : Bilet { ... }

// Oricâte interfete
class Bilet : IPretCalculabil, IValidabil, IComparable<Bilet> { ... }
```

Dacă `Bilet` ar trebui să moștenească simultan din două clase diferite, nu ar fi posibil. Dar dacă `Bilet` trebuie să respecte mai multe contracte, le poate implementa pe toate.

### Utilitatea contractului comun

Valoarea reală a interfețelor apare atunci când vrei să tratezi uniform obiecte care nu sunt legate prin moștenire, sau când vrei să decuplezi codul de tipurile concrete cu care lucrează.

```csharp
List<IPretCalculabil> bilete = new List<IPretCalculabil> { b1, b2, b3 };

double total = 0;
foreach (IPretCalculabil b in bilete)
    total += b.CalculeazaPretFinal();
```

`b1`, `b2` și `b3` pot fi de orice tip, atâta timp cât implementează `IPretCalculabil`. Codul care calculează totalul nu importă și nu cunoaște tipurile concrete. Dacă adaugi mâine `BiletCopil` care implementează `IPretCalculabil`, același `foreach` funcționează fără nicio modificare.

Același principiu face ca `CasaBilete` să poată lucra cu orice tip de bilet viitor, nu doar cu cele existente la momentul scrierii sale.

### O interfață poate fi implementată de clase fără legătură

Spre deosebire de moștenire, interfața nu impune o relație ierarhică. Clase complet diferite, fără nicio legătură structurală, pot implementa aceeași interfață și pot fi tratate uniform prin tipul ei:

```csharp
interface IValidabil
{
    bool EsteValid();
}

class Bilet : IValidabil
{
    public bool EsteValid() { return ExpiraLa > DateTime.Now; }
}

class ContUtilizator : IValidabil
{
    public bool EsteValid() { return !EsteBlockat && DataExpirareAbonament > DateTime.Now; }
}

class CupoanReducere : IValidabil
{
    public bool EsteValid() { return DataExpirare > DateTime.Now && !AFostFolosit; }
}
```

`Bilet`, `ContUtilizator` și `CupoanReducere` nu au nicio legătură prin moștenire. Dar toate pot fi validate și pot fi tratate uniform prin tipul `IValidabil`:

```csharp
List<IValidabil> entitati = new List<IValidabil> { bilet, cont, cupon };
foreach (IValidabil e in entitati)
{
    if (!e.EsteValid())
        Console.WriteLine("Entitate invalida.");
}
```

Codul de validare funcționează identic pentru toate tipurile, fără să știe nimic despre implementările concrete.

### Interfețele și clasele derivate

O clasă derivată moștenește și implementările de interfețe ale clasei de bază. `BiletStudent` moștenește `Bilet`, deci implementează automat `IPretCalculabil` și `IValidabil` și nu este nevoie să le declare explicit din nou.

Dacă `BiletStudent` suprascrie metodele cu `override`, **versiunile suprascrise sunt cele apelate** atunci când accesul se face prin tipul interfeței. Polimorfismul funcționează la fel, indiferent dacă variabila este de tipul clasei de bază sau de tipul interfeței:

```csharp
BiletStudent bs = new BiletStudent(...);

IPretCalculabil pc = bs;
Console.WriteLine(pc.GetReducere());   // apeleaza BiletStudent.GetReducere(), nu Bilet.GetReducere()

IValidabil iv = bs;
Console.WriteLine(iv.EsteValid());     // apeleaza BiletStudent.EsteValid() dacă este suprascrisă
```

Interfețele nu schimbă regulile de polimorfism, ci le extind la un nivel suplimentar de abstractizare.


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-2-programarea-orientata-obiect-in-c/interfete.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
