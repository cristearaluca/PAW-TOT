# EventArgs

Un eveniment nu este doar o notificare de tipul „s-a întâmplat ceva". În practică, handler-ul care primește notificarea are nevoie de context pentru a putea acționa în mod util. Când o comandă își schimbă starea, handler-ul vrea să știe: care comandă, din ce stare, în ce stare nouă. Când o comandă este livrată, vrea să știe data livrării și, eventual, costul de transport.

Fără un mecanism de transmitere a datelor, singura alternativă ar fi ca handler-ul să interogheze el însuși obiectul sursă după primirea notificării. Aceasta introduce un cuplaj suplimentar și poate produce inconsistențe dacă starea obiectului s-a modificat între momentul declanșării evenimentului și momentul interogării.

Soluția standard în C# este transmiterea datelor direct prin eveniment, ca parte a notificării. Mecanismul pentru aceasta este **clasa `EventArgs`** și clasele derivate din ea.

### Clasa `EventArgs`

`EventArgs` este o clasă definită în `System`, din care derivă toate clasele de date asociate evenimentelor. În sine nu conține proprietăți utile, ci există ca **punct de extensie**, un contract care spune: „orice clasă derivată din mine poate fi transmisă ca al doilea argument al unui handler de eveniment".

Există și o instanță specială, `EventArgs.Empty`, care se folosește când un eveniment nu transportă nicio dată suplimentară:

```csharp
public event EventHandler DepozitInchis;

// Declansare fara date suplimentare
DepozitInchis?.Invoke(this, EventArgs.Empty);
```

Când evenimentul are date specifice, se creează o clasă derivată din `EventArgs` cu proprietățile relevante.

### Crearea claselor `EventArgs` personalizate

Convenția de denumire este `NumeEvenimentEventArgs` (numele evenimentului urmat de sufixul `EventArgs`). Clasa nu conține logică: este un **container de date** transmis de la emițătorul evenimentului la handler-ele abonate.

```csharp
// Date transmise la fiecare schimbare de stare a comenzii
class ComandaSchimbatStareEventArgs : EventArgs
{
    public Comanda Comanda { get; set; }
    public StareComanda StareVeche { get; set; }
    public StareComanda StareNoua { get; set; }
}

// Date transmise cand comanda ajunge in starea Livrata
class ComandaLivrataEventArgs : EventArgs
{
    public Comanda Comanda { get; set; }
    public DateTime DataLivrare { get; set; }
}
```

Fiecare clasă conține exact proprietățile relevante pentru contextul evenimentului respectiv. `ComandaSchimbatStareEventArgs` transmite atât starea veche, cât și starea nouă. Handler-ul nu trebuie să mai interogheze obiectul `Comanda` pentru a afla tranziția completă. `ComandaLivrataEventArgs` adaugă data livrării, informație care poate fi diferită de momentul apelului handler-ului dacă procesarea este asincronă.

### Legătura dintre `EventArgs` și declararea evenimentului

Tipul `EventArgs` personalizat este specificat ca parametru generic al lui `EventHandler<T>`:

```csharp
class Depozit
{
    public string NumeDepozit { get; set; }
    private List<Comanda> comenzi = new List<Comanda>();

    public event EventHandler<ComandaSchimbatStareEventArgs> ComandaSchimbatStare;
    public event EventHandler<ComandaLivrataEventArgs> ComandaLivrata;

    protected virtual void OnComandaSchimbatStare(ComandaSchimbatStareEventArgs e)
    {
        ComandaSchimbatStare?.Invoke(this, e);
    }

    protected virtual void OnComandaLivrata(ComandaLivrataEventArgs e)
    {
        ComandaLivrata?.Invoke(this, e);
    }
}
```

Fiecare eveniment are propriul tip `EventArgs`, ceea ce înseamnă că fiecare are o semnătură de handler distinctă. Compilatorul garantează că handler-ul abonat la `ComandaSchimbatStare` primește un `ComandaSchimbatStareEventArgs`, nu un `ComandaLivrataEventArgs`, astfel încât nu există risc de confuzie între tipuri la runtime.

### Construirea obiectului `EventArgs` la declanșare

Clasa de date este instanțiată în metoda care declanșează evenimentul, cu toate proprietățile completate la momentul declanșării:

```csharp
public void AvansezaStare(string numarComanda)
{
    Comanda comanda = comenzi.Find(c => c.NumarComanda == numarComanda);
    if (comanda == null) return;

    StareComanda stareVeche = comanda.Stare;
    comanda.Stare = stareVeche.Urmatoarea();  // logica de tranzitie

    // Construim argumentele cu datele complete ale tranzitiei
    OnComandaSchimbatStare(new ComandaSchimbatStareEventArgs
    {
        Comanda = comanda,
        StareVeche = stareVeche,
        StareNoua = comanda.Stare
    });

    // Daca starea noua este Livrata, declansam si evenimentul specific
    if (comanda.Stare == StareComanda.Livrata)
    {
        OnComandaLivrata(new ComandaLivrataEventArgs
        {
            Comanda = comanda,
            DataLivrare = DateTime.Now
        });
    }
}
```

Obiectul `EventArgs` este creat o singură dată, cu toate datele disponibile la momentul declanșării, și este transmis tuturor handler-elor abonate. Handler-ele primesc o imagine consistentă a stării la momentul evenimentului - nu trebuie să interogheze obiectul sursă și nu există riscul ca starea să se fi modificat între timp.

### Scrierea handler-elor

Handler-ele respectă semnătura `(object sender, TEventArgs e)` și accesează datele prin parametrul `e`:

```csharp
void OnComandaSchimbatStare(object sender, ComandaSchimbatStareEventArgs e)
{
    // Accesam sursa evenimentului
    Depozit depozit = sender as Depozit;

    Console.WriteLine($"[{depozit?.NumeDepozit}] Comanda {e.Comanda.NumarComanda}: " +
                      $"{e.StareVeche} -> {e.StareNoua}");
}

void OnComandaLivrata(object sender, ComandaLivrataEventArgs e)
{
    Console.WriteLine($"Comanda {e.Comanda.NumarComanda} livrata " +
                      $"la {e.DataLivrare:dd.MM.yyyy HH:mm}");
}
```

Parametrul `sender` este convertit la tipul concret al sursei cu operatorul `as`. Dacă conversia eșuează (sursa nu este de tipul așteptat), `as` returnează `null` în loc să arunce excepție, ceea ce permite tratarea gracioasă a cazurilor neașteptate.

### Abonarea și observarea completă

Piesele se asamblează într-un flux complet:

```csharp
Depozit depozit = new Depozit { NumeDepozit = "Depozit Central" };

// Abonare cu metode denumite
depozit.ComandaSchimbatStare += OnComandaSchimbatStare;
depozit.ComandaLivrata += OnComandaLivrata;

// Abonare suplimentara cu lambda — jurnal concis
depozit.ComandaSchimbatStare += (sender, e) =>
    Console.WriteLine($"  [auto] {e.Comanda.NumarComanda} este acum {e.StareNoua}");

// Inregistrare comenzi
depozit.InregistreazaComanda(new Comanda { NumarComanda = "CMD-001", NumeClient = "Ion Popescu" });

// Avansam comanda prin stari — evenimentele se declanseaza automat
depozit.AvansezaStare("CMD-001");  // Plasata -> Procesata
depozit.AvansezaStare("CMD-001");  // Procesata -> Expediata
depozit.AvansezaStare("CMD-001");  // Expediata -> Livrata
// La ultima avansare se declanseaza atat ComandaSchimbatStare cat si ComandaLivrata
```

Fiecare apel la `AvansezaStare` declanșează evenimentul `ComandaSchimbatStare`. Când comanda ajunge în starea `Livrata`, se declanșează și `ComandaLivrata`. Toți abonații la fiecare eveniment primesc notificarea, cu datele complete ale tranzitiei, fără să fie nevoie să cunoască logica internă a clasei `Depozit`.

### Dezabonarea și limitările lambda-urilor anonime

Dezabonarea cu `-=` necesită aceeași referință folosită la abonare. Metodele denumite pot fi dezabonate oricând:

```csharp
depozit.ComandaSchimbatStare -= OnComandaSchimbatStare;
// De acum, OnComandaSchimbatStare nu mai primeste notificari
// Lambda-ul anonim continua sa primeasca
```

Lambda-urile adăugate fără stocare într-o variabilă nu pot fi dezabonate ulterior, deoarece nu există nicio referință prin care să poată fi identificate. Dacă dezabonarea va fi necesară, lambda-ul trebuie stocat:

```csharp
// Lambda stocat — poate fi dezabonat
EventHandler<ComandaSchimbatStareEventArgs> jurnalAuto = (sender, e) =>
    Console.WriteLine($"  [auto] {e.Comanda.NumarComanda} este acum {e.StareNoua}");

depozit.ComandaSchimbatStare += jurnalAuto;
// ... mai tarziu, cand nu mai e nevoie ...
depozit.ComandaSchimbatStare -= jurnalAuto;
```


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-3-delegati-actiuni-si-evenimente/eventargs.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
