# Action, Func, Predicate

Delegații personalizați (declarați cu `delegate` la nivel de namespace) sunt uneori necesari și justificați, în special atunci când numele tipului adaugă claritate semantică unui API public. Există, însă, o problemă care apare rapid în proiecte reale: multe semnături de metode se repetă structural, chiar dacă contextele sunt diferite.

Considerați următoarele declarații care ar putea exista în același proiect:

```csharp
delegate void NotificareClient(string numarComanda, string mesaj);
delegate void LogHandler(string numarComanda, string mesaj);
delegate void AlertaHandler(string numarComanda, string mesaj);
```

Toate trei descriu exact aceeași semnătură: două `string`-uri ca parametri, fără valoare returnată. Cu toate acestea, din perspectiva compilatorului, sunt **tipuri distincte și incompatibile**. O variabilă de tip `NotificareClient` nu poate primi o metodă atribuită printr-o variabilă de tip `LogHandler`, chiar dacă semnăturile sunt identice. Această incompatibilitate structurală este o sursă de fricțiune inutilă.

.NET rezolvă această problemă prin trei familii de tipuri delegat **generice și predefinite**: `Action`, `Func` și `Predicate`. Acoperă aproape toate cazurile uzuale fără declarații personalizate și elimină redundanța descrisă mai sus.

### `Action` - metode fără valoare de retur

`Action` reprezintă orice metodă care returnează `void`. Este un tip generic cu variante pentru orice număr de parametri, de la zero la șaisprezece:

```csharp
Action              // void fara parametri
Action<T>           // void cu un parametru de tip T
Action<T1, T2>      // void cu doi parametri
Action<T1, T2, T3>  // void cu trei parametri
// ... pana la Action<T1, ..., T16>
```

Tipul `T`, `T1`, `T2` etc. sunt parametri de tip generici. Compilatorul îi substituie cu tipurile concrete specificate la utilizare. Prin urmare, `Action<string, string>` este un tip delegat care descrie orice metodă care primește doi parametri de tip `string` și returnează `void`.

Tipul `NotificareClient` definit anterior (`delegate void NotificareClient(string, string)`) este echivalent funcțional cu `Action<string, string>`. Cele două sunt interschimbabile din perspectiva comportamentului, dar nu din perspectiva tipului: compilatorul le consideră tipuri diferite și nu permite atribuirea directă între ele fără conversie.

Utilizarea `Action` în locul unui tip personalizat:

```csharp
// In loc de: NotificareClient notificator = NotificareEmail;
Action<string, string> notificator = NotificareEmail;

notificator += NotificareSMS;
notificator += (nr, msg) => Console.WriteLine("[PUSH] " + nr + ": " + msg);

notificator("CMD-001", "Comanda expediata.");
```

Rezultatul este identic cu varianta bazată pe tipul personalizat. Avantajul este că nu mai este nevoie de o declarație `delegate` separată.

### `Func` - metode cu valoare de retur

`Func` reprezintă orice metodă care returnează o valoare. Prin convenție, **ultimul parametru de tip din lista generică este întotdeauna tipul returnat**:

```csharp
Func<TResult>              // fara parametri, returneaza TResult
Func<T, TResult>           // un parametru T, returneaza TResult
Func<T1, T2, TResult>      // doi parametri, returneaza TResult
// ... pana la Func<T1, ..., T16, TResult>
```

Tipul `StrategieLivrare` definit anterior (`delegate double StrategieLivrare(Comanda)`) este echivalent cu `Func<Comanda, double>`: o metodă care primește un obiect `Comanda` și returnează un `double`.

Utilizarea `Func` pentru strategia de livrare:

```csharp
Func<Comanda, double> calculeazaLivrare;

// Strategia 1: livrare standard — 15 RON fix
calculeazaLivrare = comanda => 15.0;

// Strategia 2: gratuita pentru comenzi peste 200 RON
calculeazaLivrare = comanda =>
{
    if (comanda.GetValoareTotala() > 200)
        return 0;
    return 20.0;
};

// Strategia 3: express — 10% din valoare, minim 25 RON
calculeazaLivrare = comanda =>
{
    double cost = comanda.GetValoareTotala() * 0.10;
    return cost < 25.0 ? 25.0 : cost;
};
```

Variabila `calculeazaLivrare` poate fi reatribuită oricând cu o altă implementare. Codul care utilizează delegatul pentru a calcula costul (`calculeazaLivrare(comanda)`) rămâne neschimbat. Aceasta este esența **design pattern-ului Strategy**: comportamentul unui algoritm este substituit la runtime fără a modifica codul care îl consumă.

### `Predicate` - condiții

`Predicate<T>` este un tip delegat predefinit pentru metode care testează o condiție: primesc un parametru de tip `T` și returnează `bool`. Este echivalent structural cu `Func<T, bool>`, dar cu un nume mai expresiv în contextul operațiilor de filtrare.

```csharp
Predicate<Comanda> criteriu = comanda => comanda.GetValoareTotala() > 150;
```

Tipul `Predicate<T>` apare în metodele clasice ale lui `List<T>`: `FindAll`, `Find`, `Exists`, `RemoveAll`. Aceste metode acceptă un `Predicate<T>` și operează pe elementele listei care satisfac condiția:

```csharp
// Filtrare dupa valoare
criteriu = comanda => comanda.GetValoareTotala() > 150;
List<Comanda> comenziScumpe = depozit.Filtreaza(criteriu);

// Filtrare dupa client — closure care captureaza variabila externa
string clientCautat = "Ion Popescu";
criteriu = comanda => comanda.NumeClient == clientCautat;
List<Comanda> comenziClient = depozit.Filtreaza(criteriu);

// Filtrare dupa stare
criteriu = comanda => comanda.Stare == StareComanda.Expediata;
List<Comanda> expediate = depozit.Filtreaza(criteriu);
```

### Diferența dintre `Predicate<T>` și `Func<T, bool>`

Deși `Predicate<T>` și `Func<T, bool>` descriu aceeași semnătură, compilatorul le tratează ca **tipuri distincte**. Nu există o conversie implicită între ele, iar API-urile care cer unul nu acceptă celălalt:

```csharp
Predicate<Comanda> p = comanda => comanda.GetValoareTotala() > 150;
Func<Comanda, bool> f = comanda => comanda.GetValoareTotala() > 150;

// List<T>.FindAll() accepta Predicate<T>
comenzi.FindAll(p);   // corect
comenzi.FindAll(f);   // eroare de compilare

// LINQ Where() accepta Func<T, bool>
comenzi.Where(f);     // corect
comenzi.Where(p);     // eroare de compilare
```

Această asimetrie este o consecință istorică: `Predicate<T>` a fost introdus în .NET 2.0, odată cu genericele, ca parte a API-ului lui `List<T>`. LINQ a apărut în .NET 3.5 și a ales în mod consecvent `Func<T, bool>` pentru consistență cu restul funcțiilor de ordin superior. Ambele variante produc același comportament la runtime; alegerea depinde de API-ul pe care îl utilizezi.

### Când să declari un tip delegat personalizat

Cu `Action`, `Func` și `Predicate` disponibile, declararea unui tip delegat personalizat rămâne justificată în situații specifice.

**Când numele tipului adaugă claritate semantică.** Un parametru de tip `NotificareClient` comunică intenția mai clar decât `Action<string, string>`. Când tipul delegat apare în semnăturile publice ale unor clase sau interfețe, un nume semantic îmbunătățește lizibilitatea și documentarea automată a API-ului.

**Când tipul este utilizat în contextul evenimentelor.** Evenimentele folosesc convențional tipuri delegat cu semnătura `(object sender, TEventArgs e)`. Deși `EventHandler<T>` acoperă acest caz, există situații în care un tip personalizat este mai potrivit.

**Când tipul apare repetat în API-ul public al unui proiect.** Dacă același tip de comportament apare în zeci de locuri, un tip cu nume propriu reduce riscul de confuzie și facilitează refactorizarea.

Ca regulă practică pentru cod nou: utilizează `Action` pentru metode fără retur, `Func` pentru metode cu retur, `Predicate<T>` când lucrezi cu metodele clasice ale lui `List<T>`, și declară un tip personalizat numai când numele său adaugă valoare semantică vizibilă.


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-3-delegati-actiuni-si-evenimente/action-func-predicate.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
