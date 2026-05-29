# C# vs C++ - șiruri de caractere

În C++, lucrul cu șiruri de caractere a fost întotdeauna o sursă de complexitate. Aveți la dispoziµie `char*` (pointeri la caractere, cu gestionare manual a memoriei) și `std::string` (o clasă care gestionează memoria automat, dar este mutabil). În C#, tipul `string` este un tip referință , dar cu un comportament special: este imutabil. Aceasta înseamnă că orice operație care pare să modifice un șir (concatenare, înlocuire, etc.) creează de fapt un nou obiect `string` în memorie. Șirul original rămâne neschimbat.

```csharp
// Declarare si initializare
string nume = "Ana";
string prenume = " Popescu ";

// Concatenare clasica
string numeFull = nume + " " + prenume ;

// String interpolation ( recomandat ) - prefixul $
string salut = $"Buna ziua, {nume} {prenume}!";
Console.WriteLine(salut); // Buna ziua , Ana Popescu !

// Interpolari complexe
int varsta = 22;
double medie = 9.75;
Console.WriteLine($"{nume} are {varsta} ani si media {medie:F2}") ;

// Metode utile
Console.WriteLine(nume.Length); // 3
Console.WriteLine(nume.ToUpper()) ; // ANA
Console.WriteLine(numeFull.Contains("Pop")); // True
Console.WriteLine(numeFull.Replace("Ana", "Maria")); // Maria Popescu

// Imutabilitate - sirul original nu se modifica
string original = "test";
string modificat = original.ToUpper();
Console.WriteLine(original); // " test " - neschimbat !
Console.WriteLine(modificat); // " TEST "

```


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-1-introducere-in-c-si-.net/c-vs-c++-siruri-de-caractere.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
