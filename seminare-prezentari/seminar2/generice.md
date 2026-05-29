# Generice

Să presupunem că vrem să numărăm câte bilete de fiecare tip există în `CasaBilete`. O primă abordare produce metode aproape identice:

```csharp
public int GetNumarBileteStudent()
{
    int contor = 0;
    foreach (Bilet b in bileteCumparate)
        if (b is BiletStudent)
            contor++;
    return contor;
}

public int GetNumarBileteSenior()
{
    int contor = 0;
    foreach (Bilet b in bileteCumparate)
        if (b is BiletSenior)
            contor++;
    return contor;
}
```

Codul este identic în ambele metode, cu excepția tipului verificat. Dacă adăugăm `BiletCopil`, scriem a treia metodă identică. Dacă descoperim un bug în logica de numărare, trebuie corectat în toate metodele. Dacă decidem să schimbăm implementarea (de exemplu, să folosim LINQ în loc de buclă),  modificăm toate metodele separat.

**Genericele** rezolvă exact această situație: scriem logica o singură dată și lăsăm **tipul ca parametru**.

### Parametrul de tip generic

Un parametru de tip generic este un **placeholder pentru un tip concret**, specificat la momentul apelului. Se declară între paranteze unghiulare `<T>` după numele metodei sau al clasei. `T` este o convenție de denumire; poate fi orice identificator valid:

```csharp
public int GetNumarBiletePerTip<T>()
{
    int contor = 0;
    foreach (Bilet b in bileteCumparate)
        if (b is T)
            contor++;
    return contor;
}
```

La apel, `T` este înlocuit cu tipul concret dorit:

```csharp
int nrStudenti = casa.GetNumarBiletePerTip<BiletStudent>();
int nrSeniori  = casa.GetNumarBiletePerTip<BiletSenior>();
int nrVIP      = casa.GetNumarBiletePerTip<BiletVIP>();
```

Compilatorul generează câte o versiune a metodei pentru fiecare tip cu care este apelată. La nivel de cod scris, există o singură metodă. La nivel de execuție, fiecare apel cu un tip diferit are propria versiune specializată, generată automat.

### Constrângerea `where`

Versiunea de mai sus funcționează, dar are o limitare: compilatorul nu știe nimic despre `T`. Nu poate verifica dacă `T` are anumite proprietăți sau metode, deoarece `T` ar putea fi orice tip din întreg universul C# — un `string`, un `int`, o clasă definită de un alt programator.

Constrângerea `where` rezolvă aceasta: îi spunem compilatorului ce știm cu certitudine despre `T`:

```csharp
public int GetNumarBiletePerTip<T>() where T : Bilet
{
    return bileteCumparate.OfType<T>().Count();
}
```

`where T : Bilet` înseamnă: `T` trebuie să fie `Bilet` sau o clasă derivată din `Bilet`. Compilatorul știe acum că `T` are toate proprietățile și metodele unui `Bilet` și poate verifica codul în consecință.

Cu această constrângere, un apel greșit este respins la compilare:

```csharp
casa.GetNumarBiletePerTip<string>();   // eroare de compilare: string nu este derivat din Bilet
casa.GetNumarBiletePerTip<int>();      // eroare de compilare: int nu este derivat din Bilet
casa.GetNumarBiletePerTip<BiletVIP>(); // corect: BiletVIP este derivat din Bilet
```

Erorile de tip sunt detectate la compilare, nu la runtime, ceea ce este exact comportamentul dorit.

### Clase generice

Genericele nu se limitează la metode. O clasă întreagă poate fi parametrizată cu unul sau mai mulți parametri de tip. Cel mai cunoscut exemplu din C# este `List<T>`:

```csharp
List<Bilet>  bilete = new List<Bilet>();
List<string> nume   = new List<string>();
List<int>    numere = new List<int>();
```

`List<T>` este o singură clasă scrisă o singură dată. Compilatorul generează versiuni specializate pentru `Bilet`, `string`, `int` și orice alt tip cu care o instanțiezi. Fără generice, am fi nevoiți să avem `ListaBilete`, `ListaStrings`, `ListaNumere`, care ar reprezenta clase separate cu cod identic și fără siguranța tipurilor.

### Constrângeri disponibile

O metodă sau clasă generică poate impune mai multe constrângeri simultan, separate prin virgulă. C# oferă mai multe forme de constrângere:

```csharp
where T : class          // T trebuie să fie tip referință (clasă, interfață, delegat)
where T : struct         // T trebuie să fie tip valoare (struct, int, etc.)
where T : new()          // T trebuie să aibă constructor fără parametri
where T : NumeClasa      // T trebuie să fie NumeClasa sau o clasă derivată
where T : NumeInterfata  // T trebuie să implementeze interfața respectivă
```

Pot fi combinate:

```csharp
public void Proceseaza<T>() where T : Bilet, IValidabil
{
    // T trebuie să fie Bilet sau derivat, ȘI să implementeze IValidabil
}
```

Constrângerile multiple se aplică simultan. `T` trebuie să satisfacă toate condițiile declarate. Compilatorul verifică fiecare apel și respinge tipurile care nu respectă constrângerile.

### Genericele și LINQ - combinarea cu `OfType`

Metoda `GetNumarBiletePerTip<T>` ilustrează o combinare naturală între generice și LINQ. `OfType<T>()` din LINQ este ea însăși o metodă generică, iar combinarea cu o metodă generică proprie produce cod concis și type-safe:

```csharp
public int GetNumarBiletePerTip<T>() where T : Bilet
{
    return bileteCumparate.OfType<T>().Count();
}
```

Apelantul specifică tipul, compilatorul verifică că tipul respectă constrângerea `where T : Bilet`, iar `OfType<T>()` filtrează colecția la runtime după tipul real al obiectelor. Toate cele trei straturi — verificarea la compilare, filtrarea la runtime și agregarea — sunt exprimate pe o singură linie.


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-2-programarea-orientata-obiect-in-c/generice.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
