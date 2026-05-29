# C# vs C++ - tipuri nulabile

În C++, un `int` are întotdeauna o valoare. Dacă doriți să reprezentați lipsa unei valori,trebuie să folosiți convenții (de exemplu, −1 ca valoare sentinel) sau `std::optional` (din C++17). C# oferă suport nativ pentru tipuri nullable prin sufixul `?`. Un `int?` poate stoca fie un întreg, fie valoarea specială `null`.

```csharp
int? notaExamen = null; // studentul nu a dat inca examenul

if (notaExamen.HasValue)
    Console.WriteLine($"Nota: {notaExamen.Value}");
else
    Console.WriteLine("Examenul nu a fost sustinut.");

// Operatorul ?? (null-coalescing) - ofera o valoare implicita
int notaFinala = notaExamen ?? 0; // daca e null , foloseste 0
Console.WriteLine($"Nota finala: {notaFinala}");

// Operatorul ?. (null-conditional) - acces sigur la membri
string text = null;
int? lungime = text?.Length; // nu arunca exceptie, returneaza null
Console.WriteLine(lungime ?? -1); // afiseaza -1
```


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-1-introducere-in-c-si-.net/c-vs-c++-tipuri-nulabile.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
