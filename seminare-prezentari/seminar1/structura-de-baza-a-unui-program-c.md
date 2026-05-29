# Structura de bază a unui program C\#

După crearea unui proiect nou, Visual Studio va genera automat un fișier `Program.cs` cu următorul conținut:

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace Seminar1
{
    internal class Program
    {
        static void Main(string[] args)
        {
        }
    }
}
```

Observăm câteva elemente fundamentale:

* directiva `using System;`  de la începutul fișierului importă namespace-ul `System`, care conține clase fundamentale precum `Console`. În C++, această directivă ar fi echivalentul `#include<iostream>` urmat de `using namespace std;`.&#x20;
* conceptele de `class`, `static`, `void` , `namespace` sau `string`  coincid ca intenție cu echivalentul lor din C++.
* în C# există și alți modificatori de acces, pe lângă `public`, `private` și `protected`. Unul dintre ei este `internal`, despre care vom discuta într-o secțiune dedicată.

Un aspect important este reprezentat de faptul că, spre deosebire de C++, în C# nu există posibilitatea definirii funcțiilor libere. Orice funcție (metodă) trebuie să aparțină unei clase.


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-1-introducere-in-c-si-.net/structura-de-baza-a-unui-program-c.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
