# Rezumat

### Delegații

Un delegat este un tip care descrie semnătura unei metode. O variabilă de tip delegat stochează o referință către o metodă compatibilă și poate fi invocată ca și cum ar fi metoda respectivă.

```csharp
delegate void NotificareClient(string numarComanda, string mesaj);

static void NotificareEmail(string nr, string msg) { Console.WriteLine("[EMAIL] " + nr + ": " + msg); }

NotificareClient notificator = NotificareEmail;
notificator("CMD-001", "Comanda plasata.");
```

| Concept                  | Ce înseamnă                                                  |
| ------------------------ | ------------------------------------------------------------ |
| **`delegate`**           | Cuvântul cheie pentru declararea unui tip delegat            |
| **Atribuire**            | Stochează referința către metodă, nu o apelează              |
| **Invocare**             | Apelul efectiv al metodei stocate                            |
| **Delegat ca parametru** | Permite transmiterea comportamentului ca argument            |
| **`?.Invoke(...)`**      | Invocare sigură — nu aruncă excepție dacă delegatul e `null` |

Delegații sunt mecanismul de bază pe care se construiesc `Action`, `Func` și evenimentele.

### Metode anonime și expresii lambda

Permit definirea unui comportament direct la locul utilizării, fără o declarație separată și fără un nume.

```csharp
// Metoda anonima (C# 2.0)
NotificareClient n = delegate(string nr, string msg) { Console.WriteLine("[PUSH] " + nr + ": " + msg); };

// Lambda echivalenta (C# 3.0) — forma preferata
NotificareClient n = (nr, msg) => Console.WriteLine("[PUSH] " + nr + ": " + msg);
```

| Concept                    | Ce înseamnă                                                       |
| -------------------------- | ----------------------------------------------------------------- |
| **`delegate(...) { }`**    | Sintaxa originală pentru metode anonime                           |
| **`(p1, p2) => expresie`** | Lambda cu mai mulți parametri                                     |
| **`param => { ... }`**     | Lambda cu corp din mai multe instrucțiuni                         |
| **Closure**                | Lambda captează variabile din context ca referință, nu ca valoare |
| **Lambda și `-=`**         | Lambda anonim nu poate fi dezabonat fără o referință stocată      |

Lambda-ul captează referința la variabilă, nu valoarea ei la momentul definirii. Dacă variabila se schimbă, lambda vede noua valoare.

### `Action`, `Func` și `Predicate`

Delegați generici predefiniti în .NET care acoperă majoritatea scenariilor fără declarații custom.

```csharp
Action<string, string> notificator = NotificareEmail;
Func<Comanda, double> livrare = comanda => 15.0;
Predicate<Comanda> filtru = comanda => comanda.GetValoareTotala() > 150;
```

| Tip                | Semnătură       | Cazul de utilizare                    |
| ------------------ | --------------- | ------------------------------------- |
| `Action`           | `void ()`       | Fără parametri, fără retur            |
| `Action<T1, T2>`   | `void (T1, T2)` | Doi parametri, fără retur             |
| `Func<T, TResult>` | `TResult (T)`   | Un parametru, cu retur                |
| `Predicate<T>`     | `bool (T)`      | Condiție de filtrare pentru `List<T>` |

`Predicate<T>` și `Func<T, bool>` au aceeași semnătură dar sunt tipuri diferite. `List<T>.FindAll()` acceptă `Predicate<T>`, LINQ `Where` acceptă `Func<T, bool>`.

### Multicast

Un delegat poate stoca referințe către mai multe metode simultan. La invocare, toate sunt apelate în ordinea adăugării.

```csharp
Action<string, string> notificator = NotificareEmail;
notificator += NotificareSMS;
notificator += NotificareLog;

notificator("CMD-001", "Comanda expediata.");  // apeleaza toate trei
```

| Concept                   | Ce înseamnă                                                          |
| ------------------------- | -------------------------------------------------------------------- |
| **`+=`**                  | Adaugă o metodă la lista delegatului                                 |
| **`-=`**                  | Elimină o metodă din lista delegatului                               |
| **Ordine de invocare**    | Metodele sunt apelate în ordinea adăugării, garantat                 |
| **Valoare returnată**     | Doar valoarea ultimei metode este returnată; celelalte sunt ignorate |
| **`GetInvocationList()`** | Returnează toate metodele din listă pentru invocare manuală          |
| **`?.Invoke(...)`**       | Invocare sigură dacă lista e goală (`null`)                          |

`+=` creează un delegat nou — delegații sunt imuabili. Dacă toate metodele sunt eliminate cu `-=`, variabila devine `null`.

### Evenimente (`event`)

`event` restricționează accesul la un delegat: din exterior sunt permise doar `+=` și `-=`. Invocarea și atribuirea directă sunt rezervate clasei proprietare.

```csharp
class Depozit
{
    public event EventHandler<ComandaSchimbatStareEventArgs> ComandaSchimbatStare;

    public void AvansezaStare(string numarComanda)
    {
        // ...
        ComandaSchimbatStare?.Invoke(this, new ComandaSchimbatStareEventArgs { ... });
    }
}

depozit.ComandaSchimbatStare += OnComandaSchimbatStare;  // OK
depozit.ComandaSchimbatStare = null;                     // eroare de compilare
depozit.ComandaSchimbatStare("CMD", args);               // eroare de compilare
```

| Concept                 | Ce înseamnă                                                   |
| ----------------------- | ------------------------------------------------------------- |
| **`event`**             | Restricționează accesul la delegat din exterior               |
| **Din exterior**        | Doar `+=` și `-=` sunt permise                                |
| **Din interior**        | Clasa proprietară poate invoca și atribui direct              |
| **`?.Invoke(this, e)`** | Declanșare sigură — nu aruncă excepție dacă nu există abonați |
| **`EventHandler<T>`**   | Delegatul standard pentru evenimente cu date de tip `T`       |
| **`sender`**            | Obiectul care a declanșat evenimentul                         |

`event` adaugă encapsulare: numai clasa care deține evenimentul îl poate declanșa.

### Evenimente cu date - `EventArgs` custom

Clasa `EventArgs` custom este un container de date transmis cu evenimentul.

```csharp
class ComandaSchimbatStareEventArgs : EventArgs
{
    public Comanda Comanda { get; set; }
    public StareComanda StareVeche { get; set; }
    public StareComanda StareNoua { get; set; }
}

class ComandaLivrataEventArgs : EventArgs
{
    public Comanda Comanda { get; set; }
    public DateTime DataLivrare { get; set; }
    public double CostLivrare { get; set; }
}
```

| Concept                 | Ce înseamnă                                              |
| ----------------------- | -------------------------------------------------------- |
| **`EventArgs` custom**  | Clasă derivată din `EventArgs` cu datele relevante       |
| **`EventHandler<T>`**   | Delegatul predefinit: `void Handler(object sender, T e)` |
| **`OnNumeEveniment()`** | Metodă `protected virtual` care encapsulează declanșarea |
| **`sender`**            | Convertit cu `as` la tipul real al sursei                |
| **`e`**                 | Argumentele evenimentului de tipul clasei custom         |

Pattern-ul complet de declanșare:

```csharp
protected virtual void OnComandaSchimbatStare(ComandaSchimbatStareEventArgs e)
{
    ComandaSchimbatStare?.Invoke(this, e);
}
```


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-3-delegati-actiuni-si-evenimente/rezumat.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
