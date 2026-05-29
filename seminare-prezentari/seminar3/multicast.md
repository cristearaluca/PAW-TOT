# Multicast

În capitolele anterioare, un delegat stoca o referință la exact o metodă. Invocarea delegatului apela acea metodă și producea rezultatul ei. Acest model este suficient pentru callback-uri simple și pentru pattern-ul Strategy, dar nu acoperă un scenariu frecvent în practică: mai mulți observatori care trebuie notificați simultan de același eveniment.

Imaginați-vă că la procesarea unei comenzi doriți să trimiteți un email, să scrieți în jurnal și să actualizați un dashboard, toate trei la aceeași invocare. Cu un delegat simplu, ai fi nevoit să apelezi trei delegați separați în secvență. Aceasta creează un cuplaj inutil: codul care procesează comanda trebuie să știe câți observatori există și să îi apeleze explicit pe fiecare.

C# rezolvă această problemă prin **delegați multicast**: un delegat care stochează nu una, ci o **listă ordonată de metode**. O singură invocare a delegatului apelează toate metodele din listă, în ordinea în care au fost adăugate. Această proprietate nu este o extensie opțională, ci este comportamentul implicit al oricărui delegat în C#. Fiecare variabilă de tip delegat este, prin natură, multicast.

### Adăugarea metodelor - operatorul `+=`

Metodele se adaugă în lista unui delegat cu operatorul `+=`:

```csharp
Action<string, string> notificator = NotificareEmail;

notificator += NotificareSMS;
notificator += NotificareLog;
```

La invocarea `notificator("CMD-001", "Comanda expediata.")`, runtime-ul parcurge lista internă și apelează, pe rând, `NotificareEmail`, `NotificareSMS` și `NotificareLog`, cu aceiași argumente.

```csharp
notificator("CMD-001", "Comanda expediata.");
// Output, in ordine:
// [EMAIL] CMD-001: Comanda expediata.
// [SMS] CMD-001: Comanda expediata.
// [LOG 10:30] CMD-001: Comanda expediata.
```

Este important să înțelegem ce face operatorul `+=` la nivel intern. Un delegat este **imuabil.** Odată creat, lista sa de metode nu poate fi modificată. Operatorul `+=` nu modifică obiectul delegat existent. În schimb, creează un **obiect nou** care combină lista precedentă cu noua metodă și atribuie acest obiect nou variabilei. Expresia `notificator += NotificareSMS` este sintactic echivalentă cu:

```csharp
notificator = (Action<string, string>)Delegate.Combine(notificator, NotificareSMS);
```

Implicația practică, pe care am analizat-o în capitolul despre delegați ca tip de referință, este că operatorul `+=` aplicat unei variabile nu afectează alte variabile care refereau anterior același obiect delegat.

### Eliminarea metodelor - operatorul `-=`

Operatorul `-=` elimină o metodă din lista delegatului, urmând aceeași logică de imuabilitate: creează un obiect nou care conține lista precedentă minus metoda specificată și atribuie obiectul nou variabilei.

```csharp
notificator -= NotificareSMS;
// De acum, lista contine doar NotificareEmail si NotificareLog

notificator("CMD-002", "Test 2.");
// Output:
// [EMAIL] CMD-002: Test 2.
// [LOG 10:30] CMD-002: Test 2.
```

Dacă metoda specificată nu se găsește în listă, operatorul `-=` nu produce eroare. Pur și simplu nu modifică lista.

#### Limitarea cu expresii lambda

O subtilitate importantă apare când încercăm să eliminăm un handler adăugat ca expresie lambda anonimă:

```csharp
notificator += (nr, msg) => Console.WriteLine("[PUSH] " + nr + ": " + msg);
notificator -= (nr, msg) => Console.WriteLine("[PUSH] " + nr + ": " + msg);  // nu elimina nimic
```

Deși cele două expresii lambda sunt sintactic identice, ele produc **obiecte diferite în memorie** la compilare. Operatorul `-=` caută în lista internă o referință exactă la același obiect, nu o metodă cu comportament identic. Compararea se face prin identitate de referință, nu prin egalitate structurală.

Consecința practică este că, pentru a putea elimina ulterior un handler definit ca lambda, acesta trebuie stocat într-o variabilă:

```csharp
Action<string, string> push = (nr, msg) => Console.WriteLine("[PUSH] " + nr + ": " + msg);
notificator += push;

// ... mai tarziu ...
notificator -= push;  // functioneaza — aceeasi referinta
```

Stocând lambda-ul într-o variabilă, ne asigurăm că la eliminare folosim exact aceeași referință cu care s-a făcut adăugarea.

### Ordinea de invocare

Metodele dintr-un delegat multicast sunt apelate **în ordinea în care au fost adăugate cu `+=`**. Această ordine este garantată de specificația limbajului C# și nu depinde de implementare sau de platforma de execuție.

```csharp
Action<string, string> notificator = NotificareLog;    // adaugata prima
notificator += NotificareEmail;                         // a doua
notificator += NotificareSMS;                           // a treia

notificator("CMD-001", "Comanda plasata.");
// [LOG 10:30] CMD-001: Comanda plasata.   <-- prima
// [EMAIL] CMD-001: Comanda plasata.       <-- a doua
// [SMS] CMD-001: Comanda plasata.         <-- a treia
```

### Valoarea returnată în delegații multicast

Atunci când tipul delegat are un tip de retur non-void, invocarea multicast introduce o ambiguitate: care dintre valorile returnate de cele N metode este rezultatul final? C# adoptă o regulă simplă: **se returnează exclusiv valoarea produsă de ultima metodă** din lista de invocare. Valorile returnate de toate celelalte metode sunt pierdute fără posibilitate de recuperare prin invocarea standard.

```csharp
Func<Comanda, double> strategii = comanda => 15.0;   // livrare standard
strategii += comanda => 20.0;                         // alta strategie
strategii += comanda => 0.0;                          // gratuita

double cost = strategii(comanda);
// cost == 0.0 — doar valoarea ultimei metode
// Valorile 15.0 si 20.0 sunt pierdute
```

Din acest motiv, delegații multicast cu tip de retur non-void sunt rari în practică și aproape niciodată intenționați. Situațiile legitime de multicast implică în marea majoritate a cazurilor metode cu `void`: notificări, jurnalizare, actualizări de UI.

### `GetInvocationList` - accesul la metodele individuale

Există situații în care invocarea standard nu este suficientă: vrem să colectăm valorile returnate de fiecare metodă individual, sau vrem să gestionăm excepțiile aruncate de metodele individuale fără ca o excepție să oprească apelarea celorlalte.

Metoda `GetInvocationList()`, disponibilă pe orice delegat, returnează un array de obiecte `Delegate`, câte unul pentru fiecare metodă din lista internă, în ordinea adăugării. Fiecare element poate fi invocat separat:

```csharp
Func<Comanda, double> strategii = comanda => 15.0;
strategii += comanda => comanda.GetValoareTotala() * 0.05;

foreach (Delegate d in strategii.GetInvocationList())
{
    double cost = ((Func<Comanda, double>)d)(comanda);
    Console.WriteLine("Cost calculat: " + cost + " RON");
}
```

#### Gestionarea excepțiilor în invocare manuală

Comportamentul implicit al invocării multicast standard este că, dacă una dintre metode aruncă o excepție, execuția se oprește: metodele rămase din listă nu mai sunt apelate. Uneori, aceasta este o problemă. Un handler defect nu ar trebui să blocheze celelalte handler-e.

Invocarea manuală cu `GetInvocationList()` permite tratarea excepțiilor per handler:

```csharp
foreach (Delegate d in notificator.GetInvocationList())
{
    try
    {
        ((Action<string, string>)d)("CMD-001", "Test");
    }
    catch (Exception ex)
    {
        Console.WriteLine("Eroare in handler: " + ex.Message);
        // handler-ul curent a esuat, dar continuam cu urmatorul
    }
}
```

Aceasta este abordarea robustă pentru sisteme în care handler-ele provin din surse externe sau sunt adăugate dinamic și fiabilitatea lor nu poate fi garantată.

### Delegatul multicast și valoarea `null`

Un delegat multicast poate fi `null` în două situații: dacă nicio metodă nu a fost adăugată, sau dacă toate metodele adăugate au fost eliminate cu `-=`. Eliminarea ultimei metode dintr-un delegat produce `null`, nu un delegat cu lista goală:

```csharp
Action<string, string> notificator = null;
notificator += NotificareEmail;
notificator -= NotificareEmail;  // dupa eliminare, notificator este null

// Invocarea unui delegat null arunca NullReferenceException
notificator("CMD-001", "Test");  // exceptie
```

Verificarea înainte de invocare rămâne la fel de necesară ca în cazul delegaților simpli:

```csharp
// Forma clasica
if (notificator != null)
    notificator("CMD-001", "Test");

// Forma moderna — preferata
notificator?.Invoke("CMD-001", "Test");
```

### Multicast ca fundament al evenimentelor

Delegații multicast nu sunt doar un mecanism independent. Ei constituie baza pe care este construit sistemul de evenimente din C#. Un eveniment este, în esență, un delegat multicast cu acces controlat din exterior: codul din afara clasei poate doar să se aboneze (`+=`) sau să se dezaboneze (`-=`), nu să invoce delegatul sau să suprascrie lista de metode.

Fiecare apel `+=` la un eveniment adaugă o metodă în lista multicast internă. Când evenimentul este declanșat, toate metodele din listă sunt apelate în ordinea abonării. Capitolul următor tratează evenimentele în detaliu, iar înțelegerea multicast-ului face acel capitol imediat transparent


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-3-delegati-actiuni-si-evenimente/multicast.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
