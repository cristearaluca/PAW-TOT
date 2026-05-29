# Metode anonime și expresii lambda

În capitolul anterior am văzut că un delegat stochează o referință la o metodă și că acea metodă poate fi transmisă ca argument, atribuită unei variabile sau invocată indirect. Toate exemplele foloseau metode **denumite** (metode declarate explicit cu un nume, undeva în clasă).

Această abordare funcționează corect, dar are un cost de ordin practic. De fiecare dată când vrei să transmiți un comportament simplu ca delegat, ești obligat să declari o metodă separată, cu un nume, într-un loc specific al clasei. Metoda respectivă poate fi de doar o linie, poate fi folosită o singură dată în tot programul și poate fi lipsită de orice semnificație în afara contextului imediat al utilizării. Cu toate acestea, trebuie să existe formal ca metodă denumită pentru a putea fi referită de delegat.

Un exemplu concret din contextul exercițiului:

```csharp
// Metoda declarata formal doar ca sa poata fi transmisa ca delegat
static void NotificarePush(string numarComanda, string mesaj)
{
    Console.WriteLine("[PUSH] " + numarComanda + ": " + mesaj);
}

// Undeva in Main — o singura utilizare
notificator += NotificarePush;
```

Metoda `NotificarePush` nu are nicio valoare arhitecturală în sine. Nu este un concept de domeniu, nu este reutilizată, nu contribuie la claritatea clasei. Există exclusiv pentru că sintaxa delegaților necesita, până în C# 2.0, o metodă denumită.

**Metodele anonime** și, ulterior, **expresiile lambda** elimină această constrângere: permit definirea comportamentului direct la locul utilizării, fără un nume și fără o declarație separată.

### Metode anonime

O metodă anonimă se definește cu cuvântul cheie `delegate`, urmat de lista de parametri și corpul metodei, direct în locul în care ar fi apărut numele unei metode denumite:

```csharp
NotificareClient notificator = delegate(string numarComanda, string mesaj)
{
    Console.WriteLine("[PUSH] " + numarComanda + ": " + mesaj);
};
```

Comportamentul este definit inline, exact acolo unde este nevoie. Nu există niciun nume, nu există o declarație separată în altă parte a clasei. Variabila `notificator` este inițializată direct cu un bloc de cod care respectă semnătura tipului delegat.

Metoda anonimă poate fi invocată identic cu orice alt delegat:

```csharp
notificator("CMD-001", "Comanda expediata.");
// Output: [PUSH] CMD-001: Comanda expediata.
```

Sintaxa cu `delegate(...)` este forma originală, introdusă în C# 2.0. Este mai explicită în ce privește legătura cu delegații, dar mai explicită (în engleză, verbose) decât alternativa modernă. O vei întâlni frecvent în cod scris înainte de C# 3.0 sau în proiecte cu restricții privind versiunea de limbaj, dar în cod nou este practic înlocuită de expresiile lambda.

### Expresii lambda

Expresiile lambda sunt o sintaxă mai concisă pentru exact același concept, introdusă în C# 3.0. Forma generală este:

```
(parametri) => expresie_sau_bloc
```

Semnul `=>` se pronunță în mod uzual *merge în* sau *produce*. Ce se află în stânga sunt parametrii metodei, ce se află în dreapta este corpul acesteia.

Comparând cele două forme pentru același comportament:

```csharp
// Metoda anonima — forma C# 2.0
NotificareClient notificator = delegate(string numarComanda, string mesaj)
{
    Console.WriteLine("[PUSH] " + numarComanda + ": " + mesaj);
};

// Lambda echivalenta — forma C# 3.0
NotificareClient notificator = (nr, msg) => Console.WriteLine("[PUSH] " + nr + ": " + msg);
```

Cele două sunt echivalente din perspectiva comportamentului la runtime. Diferența este exclusiv sintactică: lambda-ul elimină cuvântul cheie `delegate`, permite omiterea tipurilor parametrilor (compilatorul le inferă din tipul delegat) și permite corpul pe o singură linie fără acolade când există o singură expresie.

#### Inferența tipurilor

Un aspect important al sintaxei lambda este că **tipurile parametrilor nu trebuie specificate explicit**. Compilatorul le deduce din contextul în care lambda-ul este utilizat -mai precis, din tipul delegat declarat:

```csharp
// Tipurile sunt omise — compilatorul stie ca NotificareClient 
// asteapta (string, string), deci nr si msg sunt string
NotificareClient notificator = (nr, msg) => Console.WriteLine("[PUSH] " + nr + ": " + msg);

// Tipurile pot fi specificate explicit, dar e redundant
NotificareClient notificator2 = (string nr, string msg) => Console.WriteLine("[PUSH] " + nr + ": " + msg);
```

Forma fără tipuri explicite este cea standard în practică. Este mai concisă și la fel de clară atâta timp cât contextul este evident.

### Variante ale expresiei lambda

Sintaxa lambda suportă mai multe forme, fiecare potrivită pentru un caz diferit.

#### Un singur parametru - parantezele pot fi omise

Când lambda-ul are exact un parametru, parantezele din jurul acestuia sunt opționale:

```csharp
Action<string> log = mesaj => Console.WriteLine("[LOG] " + mesaj);

// Forma echivalenta cu paranteze
Action<string> log2 = (mesaj) => Console.WriteLine("[LOG] " + mesaj);
```

Convenția curentă în comunitatea C# este să se omită parantezele când există un singur parametru, pentru concizie.

#### Mai mulți parametri - parantezele sunt obligatorii

```csharp
NotificareClient notif = (numarComanda, mesaj) =>
    Console.WriteLine("[EMAIL] " + numarComanda + ": " + mesaj);
```

#### Fără parametri - paranteze goale

```csharp
Action alarma = () => Console.WriteLine("Depozit supraîncărcat!");
```

#### Corp cu mai multe instrucțiuni - acolade obligatorii

Când corpul lambda-ului conține mai mult de o expresie, este necesară utilizarea unui bloc cu acolade. Dacă lambda-ul are tip de retur, instrucțiunea `return` trebuie inclusă explicit:

```csharp
Func<Comanda, double> livrare = comanda =>
{
    double valoare = comanda.GetValoareTotala();
    if (valoare > 200)
        return 0;
    return 20.0;
};
```

#### Expresie simplă cu tip de retur - `return` implicit

Când corpul este o singură expresie și tipul delegat are un tip de retur non-void, valoarea expresiei devine automat valoarea returnată, fără a fi necesar `return`:

```csharp
Func<Comanda, double> livrareStandard = comanda => 15.0;
// Echivalent cu: comanda => { return 15.0; }
```

Această formă este frecvent folosită pentru funcții scurte de calcul sau transformare, unde corpul este o singură operație matematică sau o singură proiecție de date.

### Capturarea variabilelor din context

Una dintre proprietățile cele mai importante ale expresiilor lambda este că pot **accesa variabile din contextul în care sunt definite**, chiar dacă acele variabile nu sunt parametri ai lambda-ului. Acest mecanism se numește **closure** (în română, uneori tradus ca *închidere* sau *captare de context*).

Un exemplu direct:

```csharp
string canalNotificare = "EMAIL";

NotificareClient notificator = (nr, msg) =>
{
    // canalNotificare nu este parametru al lambda-ului,
    // dar este accesibila din contextul in care lambda-ul este definit
    Console.WriteLine("[" + canalNotificare + "] " + nr + ": " + msg);
};

notificator("CMD-001", "Test");  // [EMAIL] CMD-001: Test

canalNotificare = "SMS";
notificator("CMD-002", "Test");  // [SMS] CMD-002: Test
```

A doua invocare produce `[SMS]`, nu `[EMAIL]`. Aceasta ilustrează comportamentul esențial al closure-urilor: **lambda-ul capturează referința la variabilă, nu valoarea sa de la momentul definirii**. Modificările ulterioare ale variabilei sunt vizibile în lambda.

### Metode anonime față de expresii lambda

Metodele anonime (sintaxa cu `delegate(...)`) și expresiile lambda sunt, în marea majoritate a cazurilor, interschimbabile. Compilatorul le tratează identic, ambele producând un obiect delegat cu o metodă generată automat. Există totuși o diferență minoră: o metodă anonimă poate omite complet lista de parametri dacă aceștia nu sunt utilizați în corp, ceea ce nu este posibil cu sintaxa lambda:

```csharp
// Metoda anonima fara parametri declarati — permis chiar daca delegatul asteapta parametri
NotificareClient notificator = delegate
{
    Console.WriteLine("Notificare trimisa.");
};

// Echivalentul lambda — parametrii trebuie declarati (pot fi ignorati cu _)
NotificareClient notificator2 = (_, _) => Console.WriteLine("Notificare trimisa.");
```

Această diferență este rareori relevantă în practică. În cod modern, expresia lambda este forma standard; metoda anonimă rămâne utilă în principal pentru citirea codului existent.


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-3-delegati-actiuni-si-evenimente/metode-anonime-si-expresii-lambda.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
