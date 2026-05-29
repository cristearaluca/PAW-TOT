# C# vs C++ - proprietăți

În C++, practica standard pentru a controla accesul la câmpurile unei clase este de a le declara private și de a oferi metode getter și setter explicite. C# introduce conceptul de proprietăți, care combină simplitatea accesului direct la câmpuri cu controlul oferit de getter/setter, într-o sintaxă elegantă.

```csharp
class Student
{
    // Proprietate auto - implementata (cel mai frecvent folosita )
    public string Nume { get; set; }
    public int Varsta { get ; set; }
    
    // Proprietate read - only
    public string Grupa { get; private set; }
    
    // Proprietate cu logica custom
    private double nota ;
    public double Nota
    {
        get { return nota ; }
        set
        {
            if ( value < 1 || value > 10)
                throw new ArgumentException (" Nota invalida !");
            nota = value ;
        }
    }

    //Constructor
    public Student(string nume, int varsta, string grupa)
    {
        Nume = nume ;
        Varsta = varsta ;
        Grupa = grupa ;
    }
}

// Utilizare - arata ca un acces direct la camp , dar este controlat
Student s = new Student ("Ion", 21, " 1055");
s.Nume = "Maria"; // apeleaza setter -ul
Console.WriteLine(s.Nume); // apeleaza getter -ul
s.Nota = 9.5; // trece prin validare
//s.Nota = 15; // ar arunca exceptie

```


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-1-introducere-in-c-si-.net/c-vs-c++-proprietati.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
