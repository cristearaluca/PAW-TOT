# Clase și obiecte

Înainte de programarea orientată pe obiecte, un program era în esență o secvență de instrucțiuni care operau pe date separate. Datele stăteau într-un loc, funcțiile în altul, iar legătura dintre ele era responsabilitatea exclusivă a programatorului. Ușor de pierdut pe parcurs, ușor de stricat la o modificare. Orice schimbare în structura datelor implica găsirea și actualizarea tuturor funcțiilor care le prelucrau, fără niciun ajutor din partea compilatorului.

Programarea orientată pe obiecte răspunde la o întrebare practică: cum menținem împreună datele și comportamentul care aparțin aceluiași concept?

Răspunsul este **clasa**.

### Ce este o clasă?

O clasă este un **șablon.** Descrie ce date conține un lucru și ce operații poate efectua cu ele. Ea nu există în memorie ca atare; este doar o definiție, comparabilă cu un plan de construcție sau cu o rețetă. Planul descrie cum arată casa, dar nu este el însuși o casă locuibilă.

În sistemul de vânzare a biletelor pe care îl construim în acest seminar, un client are un nume, un prenume, un email și un număr de telefon. Clasa `Client` exprimă exact această realitate:

```csharp
class Client
{
    public string Nume { get; set; }
    public string Prenume { get; set; }
    public string Email { get; set; }
    public string Telefon { get; set; }
}
```

Această definiție spune: orice client are aceste patru atribute. Nu există încă niciun client concret, există doar descrierea a ce înseamnă să fii client în sistemul nostru.

### Ce este un obiect?

Un obiect este o **instanță concretă** a unei clase, creată în memorie la momentul execuției cu operatorul `new`. Dacă clasa este *rețeta*, obiectul este *prăjitura*: ceva tangibil, cu valori proprii, care ocupă spațiu în memorie.

```csharp
Client c1 = new Client();
c1.Nume = "Ion";
c1.Prenume = "Popescu";
c1.Email = "ion.popescu@gmail.com";
c1.Telefon = "0721000001";

Client c2 = new Client();
c2.Nume = "Maria";
c2.Prenume = "Ionescu";
c2.Email = "maria.ionescu@yahoo.com";
c2.Telefon = "0731000002";
```

`c1` și `c2` sunt două obiecte distincte, create din același șablon. Fiecare ocupă propria zonă de memorie și are propriile valori. Modificarea unui obiect nu o afectează pe cealaltă, sunt entități independente.

### Membrii unei clase

O clasă poate conține două categorii de membri, care împreună formează definiția completă a unui concept din domeniu.

**Datele** descriu starea obiectului. Se numesc câmpuri sau proprietăți și rețin valori specifice fiecărei instanțe. Doi clienți diferiți au date diferite, chiar dacă sunt create din aceeași clasă.

**Comportamentul** descrie ce poate face obiectul. Se exprimă prin blocuri de cod executabil care operează, de regulă, pe datele propriului obiect, numite **metode**.

```csharp
class Client
{
    // Date (proprietati)
    public string Nume { get; set; }
    public string Prenume { get; set; }
    public string Email { get; set; }

    // Comportament (metoda)
    public string GetNumeComplet()
    {
        return Prenume + " " + Nume;
    }
}
```

Metoda `GetNumeComplet()` nu primește niciun parametru din exterior. Ea lucrează exclusiv cu datele obiectului pe care este apelată. Aceasta este esența programării orientate pe obiecte: datele și comportamentul care le prelucrează coexistă în același loc, formând o unitate coerentă.

```csharp
Client c = new Client();
c.Nume = "Popescu";
c.Prenume = "Ion";

Console.WriteLine(c.GetNumeComplet()); // Ion Popescu
```

### De ce nu sunt suficiente variabilele simple?

La prima vedere, s-ar putea părea că variabilele individuale sunt suficiente pentru a reprezenta un client:

```csharp
string numeClient = "Ion Popescu";
string emailClient = "ion@gmail.com";
string telefonClient = "0721000001";
```

Această abordare funcționează pentru un singur client, izolat. Problemele apar de îndată ce vrei să lucrezi cu mai mulți clienți, să transmiți datele unui client ca argument la o metodă, sau să îi stochezi într-o colecție. Fără o clasă, datele unui client sunt o colecție de variabile fără nicio legătură formală între ele, fiind ușor de amestecat și de omis.

Cu o clasă, grupul de date capătă un **tip propriu**:

```csharp
// Fara clasa — trei parametri separati, ordinea poate fi confundata usor
void TrimiteEmail(string nume, string email, string telefon) { ... }

// Cu clasa — un singur parametru, clar si extensibil
void TrimiteEmail(Client client) { ... }
```

Dacă adăugăm ulterior câmpul `Adresa` la clasa `Client`, metoda `TrimiteEmail` nu se schimbă. Ea primește același `Client`, care acum conține mai multe date. Fără clasă, am fi obligați să adăugăm un parametru suplimentar și să actualizăm toate apelurile existente.

### Relația dintre clase în contextul exercițiului

Sistemul de bilete pe care îl implementăm conține mai multe clase care se referă unele pe altele. Un bilet nu este o colecție de atribute despre client alăturate unor atribute despre film. Un bilet **aparține** unui client, iar clientul este o entitate de sine stătătoare.

O primă variantă, care ignoră această distincție, ar arăta astfel:

```csharp
class Bilet
{
    public string NumeClient { get; set; }
    public string PrenumeClient { get; set; }
    public string EmailClient { get; set; }
    public string TelefonClient { get; set; }
    public string NumeFilm { get; set; }
    // ...
}
```

Această abordare funcționează tehnic, dar are un cost important: dacă același client cumpără trei bilete, datele lui se repetă de trei ori în memorie. Dacă clientul îți schimbă numărul de telefon, trebuie actualizate toate biletele. Iar dacă vrei să transmiți clientul ca argument undeva, ești nevoit să transmiți de fapt un bilet.

Soluția corectă recunoaște că un client este o entitate independentă și că un bilet conține o **referință** către acel client:

```csharp
class Bilet
{
    public Client Client { get; set; }  // referinta catre un obiect Client
    public string NumeFilm { get; set; }
    // ...
}
```

Acum un obiect `Bilet` stochează adresa unui obiect `Client` existent. Mai multe bilete pot referi același client fără duplicare. Structura codului reflectă structura realității: clientul există independent de biletele pe care le-a achiziționat.

### Cum arată obiectele în memorie?

Clasele sunt **tipuri de referință** în C#. Aceasta înseamnă că o variabilă de tip clasă nu stochează obiectul în sine, ci **adresa** din memorie la care se află obiectul. Această distincție are consecințe practice importante.

Când atribui o variabilă de tip clasă altei variabile, nu copiezi obiectul, ci adresa. Ambele variabile ajung să refere același obiect din memorie:

```csharp
Client c1 = new Client();  // c1 contine adresa obiectului, nu obiectul
Client c2 = c1;            // c2 contine aceeasi adresa — nu este o copie

c2.Nume = "Maria";
Console.WriteLine(c1.Nume); // Maria — acelasi obiect, vazut prin doua variabile
```

Modificarea făcută prin `c2` este vizibilă și prin `c1`, pentru că ambele variabile indică spre același obiect. Aceasta este o diferență fundamentală față de tipurile simple (`int`, `double`, `bool`), care se copiază la atribuire și sunt complet independente una de cealaltă.

Aceeași logică se aplică când transmiți un obiect ca argument la o metodă: nu trimiți o copie a obiectului, ci o referință la același obiect din memorie. Orice modificare făcută în interiorul metodei este vizibilă în afara ei.


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-2-programarea-orientata-obiect-in-c/clase-si-obiecte.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
