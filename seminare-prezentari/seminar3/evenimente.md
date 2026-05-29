# Evenimente

Delegații multicast, prezentați în capitolul anterior, oferă mecanismul tehnic necesar pentru a notifica mai mulți observatori la o singură invocare. Un delegat public de tip multicast funcționează, la suprafață, ca un sistem de notificări: codul extern se poate abona cu `+=`, iar la invocare toți abonații primesc notificarea.

Această abordare are însă o deficiență fundamentală de encapsulare. Un câmp public de tip delegat nu restricționează în niciun fel operațiile pe care codul extern le poate efectua asupra sa. Orice cod care are acces la obiect poate nu doar să se aboneze, ci și să invoce delegatul în mod arbitrar, să îi suprascrie întreaga listă de abonați sau să o șteargă complet:

```csharp
class Depozit
{
    // Delegat public — expus fara restrictii
    public Action<string, string> OnComandaSchimbatStare;
}

Depozit depozit = new Depozit();

depozit.OnComandaSchimbatStare += handler;             // intentionat — abonare
depozit.OnComandaSchimbatStare = null;                  // distructiv — sterge toti abonatii
depozit.OnComandaSchimbatStare("CMD-001", "mesaj");     // incorect — invocare din exterior
depozit.OnComandaSchimbatStare = alt_handler;           // incorect — inlocuieste lista
```

Operațiile de pe ultimele trei linii nu ar trebui să fie posibile din exterior. Lista de abonați este internă clasei `Depozit` și numai aceasta ar trebui să decidă când și cum se declanșează notificarea. Codul extern ar trebui să poată exclusiv să declare interesul pentru notificare (abonare) sau să renunțe la el (dezabonare).

Cuvântul cheie `event` a fost introdus în C# exact pentru a impune această restricție.

### Ce adaugă `event` față de un delegat simplu?

`event` este un **modificator aplicat unui câmp de tip delegat** care restricționează accesul din exterior la două operații: `+=` și `-=`. Orice altă operație (atribuire directă, invocare, citire) este permisă exclusiv din interiorul clasei care declară evenimentul.

Declararea unui eveniment:

```csharp
class Depozit
{
    public event Action<string, string> OnComandaSchimbatStare;
}
```

Din exterior, singurele operații compilabile sunt abonarea și dezabonarea:

```csharp
Depozit depozit = new Depozit();

depozit.OnComandaSchimbatStare += handler;    // permis — abonare
depozit.OnComandaSchimbatStare -= handler;    // permis — dezabonare

depozit.OnComandaSchimbatStare = null;                 // eroare de compilare
depozit.OnComandaSchimbatStare("CMD-001", "mesaj");    // eroare de compilare
```

Prin adăugarea unui singur cuvânt cheie, am transformat un câmp expus nesigur într-o interfață controlată. Clasa `Depozit` rămâne singurul proprietar al mecanismului de notificare. Codul extern poate doar să se declare interesat sau să renunțe la interes.

### Pattern-ul standard `EventHandler`

Deși `event` poate fi aplicat oricărui tip delegat, .NET definește o convenție standard pentru semnătura handler-elor de eveniment care a fost adoptată consistent în întregul framework:

```csharp
void NumeHandler(object sender, EventArgs e)
```

Parametrul `sender` reprezintă obiectul care a declanșat evenimentul. Parametrul `e` este un obiect care conține datele asociate evenimentului. Acesta poate fi `EventArgs.Empty` dacă evenimentul nu transmite date suplimentare, sau o instanță a unei clase derivate din `EventArgs` cu proprietăți specifice contextului.

Tipul delegat predefinit `EventHandler` respectă exact această semnătură:

```csharp
// EventHandler fara date — echivalent cu Action<object, EventArgs>
public event EventHandler DepozitInchis;

// EventHandler cu date custom
public event EventHandler<ComandaSchimbatStareEventArgs> ComandaSchimbatStare;
```

`EventHandler<TEventArgs>` este forma generică, unde `TEventArgs` este tipul clasei de date specifice evenimentului, care trebuie să derive din `EventArgs`.

#### De ce există această convenție

Convenția `(object sender, EventArgs e)` există din primele versiuni ale platformei .NET și este respectată uniform în framework: butoane, timer-e, conexiuni de rețea, sisteme de fișiere, toate urmează același pattern. Această uniformitate produce două avantaje practice. În primul rând, `sender` permite aceluiași handler să gestioneze evenimente de la mai multe surse, iar handler-ul poate inspecta `sender` pentru a determina originea notificării. În al doilea rând, `EventArgs` poate fi subclasate cu orice proprietăți noi fără a modifica semnătura handler-ului, ceea ce păstrează compatibilitatea API-ului pe termen lung.

### Declararea evenimentelor în clasă

Declararea urmează formatul unui câmp cu modificatorul `event`:

```csharp
class Depozit
{
    public string NumeDepozit { get; set; }
    private List<Comanda> comenzi = new List<Comanda>();

    // Eveniment declansat la fiecare schimbare de stare a unei comenzi
    public event EventHandler<ComandaSchimbatStareEventArgs> ComandaSchimbatStare;

    // Eveniment declansat cand comanda ajunge in starea Livrata
    public event EventHandler<ComandaLivrataEventArgs> ComandaLivrata;
}
```

Fiecare eveniment are propriul tip `EventArgs` specializat — subiect tratat în detaliu în capitolul următor. Important de reținut acum este că fiecare eveniment reprezintă un fapt semantic distinct din domeniu: schimbarea stării este un eveniment diferit de livrare, chiar dacă ambele se referă la același obiect `Comanda`.

### Declanșarea evenimentului - pattern-ul `On...`

Evenimentele se declanșează din interiorul clasei prin invocarea delegatului. Convenția standard este să se encapsuleze declanșarea într-o metodă protejată numită `On` urmată de numele evenimentului:

```csharp
class Depozit
{
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

    public void AvansezaStare(string numarComanda)
    {
        // ... gaseste comanda, calculeaza starea noua ...

        OnComandaSchimbatStare(new ComandaSchimbatStareEventArgs
        {
            Comanda = comanda,
            StareVeche = stareVeche,
            StareNoua = comanda.Stare
        });

        if (comanda.Stare == StareComanda.Livrata)
        {
            OnComandaLivrata(new ComandaLivrataEventArgs
            {
                Comanda = comanda,
                DataLivrare = DateTime.Now
            });
        }
    }
}
```

Motivul pentru care declanșarea este separată într-o metodă `protected virtual` este extensibilitatea prin moștenire. O clasă derivată din `Depozit` poate suprascrie `OnComandaSchimbatStare` pentru a adăuga logică înainte sau după declanșarea evenimentului, sau pentru a suprima declanșarea în anumite condiții, fără a modifica clasa de bază.

Utilizarea `?.Invoke(this, e)`  în locul verificării explicite cu `if` urmată de invocare este forma modernă și recomandată. `this` este transmis ca `sender`, identificând obiectul `Depozit` care a declanșat evenimentul.

### Abonarea și dezabonarea

Din codul care utilizează clasa `Depozit`, abonarea se face cu `+=` și dezabonarea cu `-=`, identic cu operatorii pe delegați:

```csharp
Depozit depozit = new Depozit("Depozit Central");

// Abonare cu metode denumite — pot fi dezabonate cu -=
depozit.ComandaSchimbatStare += OnComandaSchimbatStare;
depozit.ComandaLivrata += OnComandaLivrata;

// Abonare cu lambda — utila pentru logica simpla
depozit.ComandaSchimbatStare += (sender, e) =>
    Console.WriteLine("[AUTO] Stare noua: " + e.StareNoua);

// Procesam comenzi — evenimentele se declanseaza automat la AvansezaStare
depozit.AvansezaStare("CMD-001");

// Dezabonare
depozit.ComandaSchimbatStare -= OnComandaSchimbatStare;
```

Handler-ul scris ca metodă denumită:

```csharp
void OnComandaSchimbatStare(object sender, ComandaSchimbatStareEventArgs e)
{
    Depozit depozit = sender as Depozit;
    Console.WriteLine($"Depozit: {depozit?.NumeDepozit}");
    Console.WriteLine($"Comanda {e.Comanda.NumarComanda}: {e.StareVeche} -> {e.StareNoua}");
}
```

`sender` este de tip `object` prin convenție, dar poate fi convertit la tipul real al sursei cu operatorul `as`. Dacă același handler gestionează evenimente de la mai mulți emițători, conversia `sender as Depozit` permite identificarea sursei concrete.

### Dezabonarea și gestionarea memoriei

O consecință importantă a modului în care funcționează evenimentele privește gestionarea memoriei. Atâta timp cât un obiect este abonat la un eveniment al altui obiect, garbage collector-ul nu poate colecta obiectul abonat. Există o referință activă de la emițătorul evenimentului la handler, care menține în viață obiectul care conține handler-ul.

Aceasta poate produce **memory leaks** subtile în scenarii în care obiectele abonate au o durată de viață mai scurtă decât emițătorii lor. Practica corectă este dezabonarea explicită cu `-=` atunci când obiectul abonat nu mai are nevoie de notificări sau urmează să fie colectat.

Lambda-urile anonime adăugate fără a fi stocate într-o variabilă nu pot fi dezabonate ulterior — aceasta este o limitare importantă de reținut atunci când alegi forma de abonare:

```csharp
// Lambda stocat — poate fi dezabonat
EventHandler<ComandaSchimbatStareEventArgs> handler = (sender, e) =>
    Console.WriteLine("[AUTO] " + e.StareNoua);

depozit.ComandaSchimbatStare += handler;
// ... mai tarziu ...
depozit.ComandaSchimbatStare -= handler;  // posibil, aceeasi referinta

// Lambda anonim nestocat — nu poate fi dezabonat
depozit.ComandaSchimbatStare += (sender, e) =>
    Console.WriteLine("[AUTO] " + e.StareNoua);
// Nu exista nicio referinta prin care sa se poata face dezabonarea
```


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-3-delegati-actiuni-si-evenimente/evenimente.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
