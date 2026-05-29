# C# vs C++ - Tipuri valoare și tipuri referință

În C++, orice variabilă poate fi alocată pe stivă sau pe heap, în funcție de modul în care este declarată. În C#, distincția este mai rigidă și este determinată de tipul variabilei, nu de modul de alocare. Tipurile valoare (value types) sunt stocate direct pe stivă (sau inline în obiectul care le conține). Acestea includ toate tipurile numerice (`int`, `double`, `float` etc.), `char`, `bool`, `enum` și `struct`. Atunci când atribuiți o variabilă de tip valoare altei variabile, se face o copie a valorii.

Tipurile referință (reference types) sunt întotdeauna alocate pe heap, iar variabila stochează doar o referință (similar unui pointer) către obiectul propriu-zis. Acestea includ `class`, `string`, array-urile și `object`. La atribuire se copiază referința, nu obiectul.

```csharp
// Tipuri valoare - se copiaza valoarea
int a = 10;
int b = a; // b este o copie independenta
b = 20; // a ramane 10

// Tipuri referinta - se copiaza referinta
int [] arr1 = { 1, 2, 3 };
int [] arr2 = arr1 ; // arr2 refera acelasi array !
arr2 [0] = 99; 
```

Un concept important legat de tipurile valoare și referință este **boxing** și **unboxing**. Deoarece în C# toate tipurile derivă din `System.Object`, un tip valoare poate fi convertit implicit într-un `object` (boxing). Aceasta presupune alocarea pe heap și copierea valorii. Operația inversă (unboxing) extrage valoarea din obiectul de pe heap. Aceste operații sunt costisitoare și trebuie evitate în cod critic din punct de vedere al performanței.

```csharp
int numar = 42;
object obj = numar ; // boxing : valoarea se împachetează într-un object
int extras = (int) obj ; // unboxing : se extrage valoarea dintr-un object

// Aliasuri : int este de fapt System . Int32

```

Acest lucru poate fi contraintuitiv la început, deoarece în C++ tipurile primitive nu derivă dintr-un tip comun. În C#, toate tipurile, inclusiv cele primitive, derivă din `System.Object` , iar denumiri precum `int` sau `float` nu sunt nimic altceva decât alias-uri la clasele care implementează concret acele tipuri.

```csharp
Console.WriteLine(typeof(int)); // System . Int32
Console.WriteLine(42.GetType ()); // System . Int32
```


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-1-introducere-in-c-si-.net/c-vs-c++-tipuri-valoare-si-tipuri-referinta.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
