# Arhitectura .NET Framework

.NET Framework este o platformă de dezvoltare creată de Microsoft care oferă un mediu de execuție controlat (managed) pentru aplicații. Arhitectura sa se bazează pe câteva componente:

* **Common Language Runtime (CLR)** este motorul de execuție al platformei .NET. CLR-ul este responsabil pentru încărcarea și execuµia codului, gestionarea memoriei prin intermediul **Garbage Collector**, verficarea tipurilor la runtime, gestionarea excepțiilor și asigurarea securității codului. În esență, CLR-ul funcționează ca un strat intermediar între codul vostru și sistemul de operare.
* **Base Class Library (BCL)** reprezintă biblioteca standard a platformei .NET și conține mii de clase organizate în namespace-uri. Exemple relevante includ `System` (tipuri fundamentale), `System.Collections.Generic` (colecții generice), `System.IO` (operații cu fișiere), `System.Windows.Forms` (interfețe grafice, pe care le vom folosi extensiv în acest curs) și `System.Data` (acces la baze de date).

Procesul de compilare în .NET diferă fundamental de cel din C++. În C++, compilatorul produce direct cod mașină nativ, specific procesorului și sistemului de operare țintă. În .NET, compilarea se desfășoară în două etape.

* În prima etapă, compilatorul C# traduce codul surs în **Intermediate Language (IL)**, un limbaj intermediar independent de platformă, similar cu bytecode-ul din Java. Rezultatul este salvat într-un fișier `.exe` sau `.dll` numit **assembly**.&#x20;
* În a doua etapă, la runtime, compilatorul **Just-In-Time (JIT)** traduce codul IL în cod mașină nativ, specific procesorului pe care rulează aplicația. Această traducere se face o singură dată pentru fiecare metodă, la primul apel.

Acest model are mai multe avantaje. Codul IL este verificat înainte de execuție, ceea ce asigură siguranța tipurilor (type safety). Compilatorul JIT poate optimiza codul pentru procesorul specific pe care rulează aplicația. Iar codul managed beneficiază automat de gestionarea memoriei prin Garbage Collector, eliminand astfel categoria întreagă de buguri legate de memory leaks, dangling pointers și double delete, probleme cu care sunteți probabil familiari din C++.

{% hint style="info" %}
**Exercițiu**

Puteți examina codul IL generat folosind utilitarul ildasm.exe (IL Disassembler), inclus în Visual Studio. Navigați în Windows Explorer în folderul proiectului vostru, apoi urmați calea bin\Debug, deschideți o fereastră de Command Prompt și tastați următoarea comandă:

`ildasm Seminar1.exe`

Veți vedea structura assembly-ului, inclusiv codul IL al fiecărei metode.
{% endhint %}


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-1-introducere-in-c-si-.net/arhitectura-.net-framework.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
