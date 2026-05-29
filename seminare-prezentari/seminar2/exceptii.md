# Excepții

Orice metodă care primește date din exterior se confruntă cu aceeași realitate: nu poate controla ce i se trimite. Un preț negativ, un număr de loc în afara intervalului valid, un email fără `@` — toate sunt situații pe care codul trebuie să le trateze în mod explicit. Ignorarea lor nu le face să dispară, ci le transformă în bug-uri ascunse care se manifestă târziu și departe de cauza reală.

Există două abordări incorecte cu care programatorii se confruntă frecvent.

Prima este să **ignori problema** și să stochezi valoarea invalidă. Obiectul ajunge într-o stare incorectă, iar efectele sunt observate mult mai târziu, în alte metode, cu mesaje de eroare care nu indică sursa reală a problemei.

A doua este să **returnezi o valoare specială** care semnalează eroarea — de exemplu `-1` sau `null`. Apelantul poate ignora această valoare sau poate să nu știe că trebuie verificată, iar problema se propagă în continuare.

Abordarea corectă este să **semnalezi explicit** că ceva a mers prost și să oprești execuția curentă. Mecanismul pentru aceasta este **excepția**.

### `throw` și tipuri de excepții

Când detectăm o problemă, aruncăm o excepție cu instrucțiunea `throw`:

```csharp
public double PretBaza
{
    set
    {
        if (value <= 0)
            throw new ArgumentException("Pretul de baza trebuie sa fie strict pozitiv.");
        pretBaza = value;
    }
}
```

`throw` oprește imediat execuția metodei curente. Valoarea invalidă nu ajunge niciodată în câmpul privat. Orice cod care depindea de continuarea execuției după această linie nu va rula.

.NET oferă o ierarhie de tipuri de excepții, fiecare potrivit pentru o categorie de situații.

**`ArgumentException`** se folosește când un parametru primit din exterior are o valoare inacceptabilă. Este excepția potrivită pentru validările din proprietăți și din constructori, situații în care problema este cu datele de intrare:

```csharp
// Numarul de loc in afara intervalului 1-200
if (value < 1 || value > 200)
    throw new ArgumentException("Numarul locului trebuie sa fie intre 1 si 200.");

// Varsta sub limita pentru biletul senior
if (varstaClient < 60)
    throw new ArgumentException("Biletul Senior este disponibil doar pentru persoane de 60+ ani.");
```

**`InvalidOperationException`** se folosește când operația cerută nu este posibilă în starea curentă a obiectului, indiferent de parametri. Este excepția potrivită pentru operații la nivel de logică, în care problema nu este cu datele, ci cu contextul în care se face apelul:

```csharp
// Nu putem returna cel mai scump bilet dintr-o colectie goala
public Bilet GetBiletulCelMaiScump()
{
    if (bileteCumparate.Count == 0)
        throw new InvalidOperationException("Nu exista bilete vandute.");
    return bileteCumparate.OrderByDescending(b => b.CalculeazaPretFinal()).First();
}
```

Alegerea corectă a tipului de excepție nu este o chestiune de stil. Ea comunică exact natura problemei apelantului și ajută la diagnosticarea rapidă a erorilor.

### Propagarea excepției pe stivă de apeluri

Când `throw` rulează, excepția nu dispare, ci **urcă pe stiva de apeluri**, metodă cu metodă, până când cineva o prinde sau programul se oprește.

```
Main()
  └── new Bilet(..., pretBaza: -10, ...)
        └── constructor Bilet
              └── PretBaza = -10
                    └── set arunca ArgumentException
                          <-- exceptia urca
            <-- constructorul nu finalizeaza
       <-- obiectul nu ajunge in memorie
<-- exceptia ajunge in Main, programul se opreste daca nu este prinsa
```

Aceasta este o proprietate fundamentală a excepțiilor: dacă constructorul aruncă excepție, **obiectul nu ajunge în memorie**. Nu poți obține un `Bilet` cu preț negativ, pentru că un astfel de obiect nu poate fi creat. Obiectele invalide sunt respinse la naștere, nu descoperite mai târziu.

### Prinderea excepțiilor cu `try``/catch`&#x20;

Pentru a preveni oprirea programului, prindem excepția cu blocul `try/catch`:

```csharp
try
{
    Bilet b = new Bilet("Dune 2", 1, TipFilm.Actiune, DateTime.Now.AddDays(1), client, 15, -10);
}
catch (ArgumentException ex)
{
    Console.WriteLine($"Date invalide: {ex.Message}");
}
```

Blocul `try` conține codul care ar putea arunca excepție. Dacă o excepție apare, execuția **sare imediat** la blocul `catch` corespunzător. Proprietatea `ex.Message` conține textul transmis la `throw new ArgumentException(...)`.

Putem prinde tipuri diferite de excepții în `catch`-uri separate, fiecare tratat în mod specific:

```csharp
try
{
    casa.AdaugaBilet(bilet);
}
catch (ArgumentException ex)
{
    Console.WriteLine($"Date invalide: {ex.Message}");
}
catch (InvalidOperationException ex)
{
    Console.WriteLine($"Operatie imposibila: {ex.Message}");
}
catch (Exception ex)
{
    // prinde orice alt tip de exceptie neprevazut
    Console.WriteLine($"Eroare neasteptata: {ex.Message}");
}
```

**Ordinea contează**: tipurile mai specifice trebuie să apară înaintea celor mai generale. `Exception` este clasa de bază pentru toate excepțiile, deci un `catch (Exception)` prinde orice. Dacă apare primul, blocurile de mai jos nu vor fi niciodată atinse.

### `finally` - cod care rulează întotdeauna

Blocul `finally` rulează indiferent dacă a apărut sau nu o excepție, atât dacă `try` s-a terminat cu succes, cât și dacă a aruncat o excepție prinsă sau neprinsă. Este destinat operațiilor de curățare care trebuie să aibă loc în orice circumstanță: închiderea unui fișier, eliberarea unei conexiuni la baza de date, deblocarea unei resurse.

```csharp
try
{
    Bilet b = new Bilet(...);
    casa.AdaugaBilet(b);
}
catch (ArgumentException ex)
{
    Console.WriteLine($"Eroare: {ex.Message}");
}
finally
{
    Console.WriteLine("Executia blocului finally — ruleaza intotdeauna.");
}
```

În exercițiul de față nu avem resurse externe de eliberat, dar `finally` apare frecvent în cod real și este important să fie înțeles înainte de a întâlni situații unde absența lui produce scurgeri de resurse.

### Ce nu trebuie să facem cu excepțiile?

Excepțiile sunt un mecanism puternic, dar utilizarea lor greșită produce cod greu de înțeles și de depanat.

**Nu folosim excepții pentru flux de control normal.** Dacă vrem să verificăm dacă un bilet este valid înainte de a-l adăuga, nu aruncăm excepție și nu o prindem ca pe un `if`. Excepțiile sunt pentru situații **excepționale,** eșecuri neașteptate, date invalide sau stări imposibile. Nu sunt ramificări logice obișnuite:

```csharp
// Gresit: exceptia ca ramura logica
try
{
    casa.AdaugaBilet(bilet);
    Console.WriteLine("Bilet adaugat.");
}
catch (Exception)
{
    Console.WriteLine("Biletul nu este valid.");
}

// Corect: verificare explicita inainte de actiune
if (bilet.EsteValid())
    casa.AdaugaBilet(bilet);
else
    Console.WriteLine("Biletul nu este valid.");
```

**Nu prindem excepții și nu le ignorăm.** Un `catch` gol înghite eroarea, ascunde problema și face diagnosticarea imposibilă. Orice excepție prinsă trebuie cel puțin logată sau afișată:

```csharp
// Gresit: eroarea dispare fara urma
catch (Exception) { }

// Corect: cel putin afisam sau logam
catch (Exception ex)
{
    Console.WriteLine($"Eroare: {ex.Message}");
}
```

**Nu aruncăm excepții de tipul greșit.** Dacă problema este cu un parametru, folosim `ArgumentException`. Dacă problema este cu starea obiectului, folosim `InvalidOperationException`. Tipul excepției comunică natura erorii și ajută apelantul să scrie `catch`-uri corecte.


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-2-programarea-orientata-obiect-in-c/exceptii.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
