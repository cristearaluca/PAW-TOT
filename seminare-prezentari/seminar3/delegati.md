# Delegați

Până în acest punct al cursului, am lucrat cu tipuri care reprezintă **date**: numere întregi, șiruri de caractere, obiecte, colecții. Un `int` stochează o valoare numerică. Un obiect de tip `Comanda` stochează starea unei comenzi. Întrebarea de la care pornim în acest capitol este aparent neobișnuită: poate un tip să reprezinte nu o dată, ci o **acțiune** — adică o metodă?

Răspunsul este da, iar mecanismul care face acest lucru posibil se numește **delegat**.

Pentru a înțelege de ce acest mecanism este necesar, să pornim de la o problemă concretă de proiectare.

#### Problema comportamentului variabil

Imaginați-vă că dezvoltați un sistem de gestiune a comenzilor pentru un depozit. Sistemul procesează comenzile printr-o metodă centrală, iar la finalul procesării trebuie să notifice clientul. Notificarea poate lua mai multe forme: un email, un SMS sau o înregistrare în jurnal. Forma concretă depinde de contextul aplicației. În producție se trimit emailuri, în testare se scrie în jurnal, într-un sistem integrat se trimit SMS-uri.

O primă abordare, la care recurg frecvent cei care nu cunosc delegații, este cea bazată pe parametri de control:

```csharp
static void ProcesesazaComanda(Comanda c, string tipNotificare)
{
    // ... logica de procesare ...

    if (tipNotificare == "email")
        NotificareEmail(c.NumarComanda, "Comanda a fost procesata.");
    else if (tipNotificare == "sms")
        NotificareSMS(c.NumarComanda, "Comanda a fost procesata.");
    else if (tipNotificare == "log")
        NotificareLog(c.NumarComanda, "Comanda a fost procesata.");
}
```

Această abordare are mai multe deficiențe grave. În primul rând, metoda `ProcesesazaComanda` cunoaște explicit toate variantele de notificare existente, fiind cuplată strâns cu detalii care nu o privesc. Rolul metodei este să proceseze o comandă, nu să decidă politica de notificare. În al doilea rând, de fiecare dată când apare o variantă nouă de notificare, trebuie modificat codul din `ProcesesazaComanda`, deși logica de procesare nu s-a schimbat cu nimic. Acesta este un simptom al violării **principiului deschis-închis** (Open/Closed Principle): o componentă bine proiectată ar trebui să fie deschisă pentru extensie, dar închisă pentru modificare.

Soluția elegantă presupune ca metoda `ProcesesazaComanda` să primească **comportamentul de notificare ca argument,** adică să primească direct metoda pe care trebuie să o apeleze, fără să știe nimic despre ce face acea metodă în interior. Pentru ca acest lucru să fie posibil, avem nevoie de un tip care să descrie semnătura acelei metode. Acesta este exact rolul delegaților.

### Ce este un delegat?

Un **delegat** este un tip care descrie semnătura unei metode. Mai precis, un tip delegat specifică:

* **lista de parametri** pe care o acceptă metodele compatibile (tipurile și ordinea lor)
* **tipul valorii returnate** de metodele compatibile

O variabilă de tip delegat poate stoca o referință la orice metodă care respectă semnătura descrisă de tipul delegat respectiv. Acea variabilă poate fi ulterior invocată ca și cum ar fi direct metoda stocată.

Din perspectiva sistemului de tipuri al limbajului C#, un delegat este un **tip de referință**. La nivel de runtime, o variabilă de tip delegat stochează, intern, adresa de memorie a metodei referite, împreună cu o referință la obiectul pe care metoda urmează să fie apelată, relevant când metoda este una de instanță. Această reprezentare internă nu este direct accesibilă programatorului, dar este utilă pentru a înțelege ce se întâmplă când atribui sau invoci un delegat.

#### Delegatul față de alte tipuri de referință

Studenții care întâlnesc delegații pentru prima dată observă uneori o asemănare superficială cu interfețele: ambele permit comportament variabil, ambele permit transmiterea unui „comportament" ca argument. Diferența esențială este că o interfață descrie **un contract pentru un obiect.** Cel care o implementează este o clasă întreagă, cu stare și cu posibil mai multe metode. Un delegat descrie **semnătura unei singure metode**. Granularitatea este complet diferită: delegatul este mai fin și mai flexibil atunci când tot ce ai nevoie este o singură operație, fără stare asociată.

O altă analogie utilă este aceea cu pointerii la funcții din limbajele C și C++. Un pointer la funcție stochează adresa unei funcții și poate fi apelat. Delegatul C# face același lucru, cu o diferență crucială: este **type-safe**. Compilatorul verifică că metoda atribuită are exact semnătura descrisă de tipul delegat. Nu există conversii implicite, nu există risc de a apela o metodă cu parametrii greșiți. Orice incompatibilitate este detectată la compilare, nu la execuție.

### Declararea unui tip delegat

Un tip delegat se declară folosind cuvântul cheie `delegate`, urmat de tipul returnat, numele noului tip și lista de parametri:

```csharp
delegate void NotificareClient(string numarComanda, string mesaj);
```

Această linie declară un tip nou numit `NotificareClient`. Ea nu creează nicio variabilă și nu stochează nicio metodă. Este, în toate privințele, echivalentul unui `class` sau `enum` . Definim un tip, nu o valoare.

Orice metodă care primește doi parametri de tip `string` și returnează `void` este **compatibilă** cu tipul `NotificareClient` și poate fi stocată într-o variabilă de acest tip.

#### Unde se declară un tip delegat

Convențional, tipurile delegat se declară la nivel de namespace, alături de alte tipuri (clase, interfețe, enumerări), nu în interiorul unei clase. Aceasta reflectă natura lor: sunt tipuri de sine stătătoare, nu auxiliare ale unei clase anume.

```csharp
namespace GestiuneDepozit
{
    // Declarare la nivel de namespace — recomandat
    delegate void NotificareClient(string numarComanda, string mesaj);
    delegate double StrategieliVrare(Comanda comanda);

    class Depozit
    {
        // ...
    }
}
```

Declararea în interiorul unei clase este permisă sintactic, dar reduce vizibilitatea tipului și sugerează în mod înșelător că delegatul aparține exclusiv acelei clase.

#### Ce generează compilatorul

Când compilatorul întâlnește o declarație `delegate`, generează automat o clasă care moștenește din `System.MulticastDelegate`. Această clasă conține, printre altele, un câmp intern care stochează lista de metode referite și o metodă `Invoke` cu semnătura exactă declarată. Când apelezi un delegat în cod, compilatorul translatează apelul în `Invoke`. Aceste detalii nu sunt necesare în utilizarea curentă, dar explică de ce un delegat are comportament de obiect (poate fi `null`, poate fi transmis ca argument, poate fi comparat) și de ce invocarea sa aruncă `NullReferenceException` când nu i-a fost atribuită nicio metodă.

### Atribuirea și invocarea

Odată declarat tipul, putem crea variabile de acel tip, le putem atribui metode și le putem invoca.

#### Atribuirea

O variabilă de tip delegat se poate inițializa direct cu numele unei metode compatibile:

```csharp
static void NotificareEmail(string numarComanda, string mesaj)
{
    Console.WriteLine($"[EMAIL] Comanda {numarComanda}: {mesaj}");
}

NotificareClient notificator = NotificareEmail;
```

Este important să observăm că `NotificareEmail` este scris **fără paranteze**. Parantezele ar însemna apelarea metodei și obținerea valorii returnate. Fără paranteze, expresia desemnează metoda în sine. Compilatorul construiește un obiect delegat care referă acea metodă și îl stochează în variabila `notificator`.

Compatibilitatea la atribuire este verificată de compilator. Dacă încercăm să atribuim o metodă cu semnătură diferită, obținem o eroare la compilare, nu la execuție:

```csharp
static void MetodaIncompatibila(int x) { }
static string MetodaCuRetur(string s) { return s; }

NotificareClient n1 = MetodaIncompatibila;  // Eroare: parametri incompatibili
NotificareClient n2 = MetodaCuRetur;        // Eroare: tip de retur incompatibil
NotificareClient n3 = NotificareEmail;      // Corect
```

Această verificare statică este una dintre proprietățile esențiale ale delegaților în C# față de mecanisme similare din alte limbaje. Erorile sunt detectate cât mai devreme, în faza de compilare.

#### Invocarea

O variabilă de tip delegat se invocă exact ca o metodă obișnuită:

```csharp
notificator("CMD-001", "Comanda ta a fost plasata.");
```

La execuție, runtime-ul urmărește referința internă și apelează metoda stocată — în cazul nostru, `NotificareEmail`. Rezultatul este identic cu un apel direct la `NotificareEmail("CMD-001", "Comanda ta a fost plasata.")`.

Avantajul apare imediat ce înlocuim metoda stocată:

```csharp
static void NotificareSMS(string numarComanda, string mesaj)
{
    Console.WriteLine($"[SMS] Comanda {numarComanda}: {mesaj}");
}

notificator = NotificareSMS;
notificator("CMD-001", "Comanda ta a fost plasata.");
// Acum se executa NotificareSMS, fara nicio modificare in codul de invocare
```

Codul care invocă `notificator(...)` nu se schimbă. Comportamentul executat se schimbă prin reatribuirea variabilei.

### Transmiterea delegaților ca argumente

Puterea reală a delegaților se manifestă atunci când sunt transmiși ca argumente către alte metode. Revenind la problema din secțiunea 1.1, putem acum scrie o soluție corectă:

```csharp
static void ProcesesazaComanda(Comanda c, NotificareClient onFinalizare)
{
    // ... logica de procesare ...
    onFinalizare(c.NumarComanda, "Comanda a fost procesata.");
}
```

Metoda `ProcesesazaComanda` nu mai cunoaște și nu mai trebuie să cunoască variantele de notificare. Primește un delegat și îl invocă. Decizia despre ce metodă concretă se execută aparține apelantului:

```csharp
ProcesesazaComanda(comanda, NotificareEmail);
ProcesesazaComanda(comanda, NotificareSMS);
ProcesesazaComanda(comanda, NotificareLog);
```

Dacă mâine apare o variantă nouă — să zicem notificare prin push notification — adăugăm o metodă compatibilă și o transmitem ca argument. Metoda `ProcesesazaComanda` nu se modifică.

Această tehnică se numește în literatura de specialitate **injecție de comportament** (behavior injection) sau, mai general, **callback.** Apelantul transmite o metodă pe care apelatul o va executa la momentul potrivit. Este un pattern fundamental în proiectarea software, prezent în interfețele grafice, în programarea asincronă și în aproape orice sistem care trebuie să fie extensibil.

### Delegați cu parametri și tip de retur

Un tip delegat poate descrie orice semnătură de metodă, inclusiv metode cu mai mulți parametri sau cu tip de retur non-void.

```csharp
delegate void AlarmaHandler();                             // fara parametri, fara retur
delegate void NotificareClient(string nr, string mesaj);  // cu parametri, fara retur
delegate double StrategieliVrare(Comanda comanda);         // cu parametru si cu retur
```

Variabila de tip `StrategieliVrare` poate fi reatribuită oricând cu altă metodă compatibilă:

```csharp
static double LivrareStandard(Comanda c) => 15.0;

static double LivrareExpress(Comanda c)
    => Math.Max(c.GetValoareTotala() * 0.10, 25.0);

StrategieliVrare strategie = LivrareStandard;
double cost1 = strategie(comanda);  // returneaza 15.0

strategie = LivrareExpress;
double cost2 = strategie(comanda);  // calculat dupa formula express
```

Codul care calculează costul prin `strategie(comanda)` rămâne identic. Singura schimbare este ce metodă este stocată în variabilă. Acesta este **design pattern-ul Strategy** exprimat prin delegați: comportamentul unui algoritm este separat de contextul care îl utilizează.

### Verificarea înainte de invocare

O variabilă de tip delegat are valoarea implicită `null` atunci când nu i-a fost atribuită nicio metodă. Invocarea unui delegat `null` aruncă o excepție de tip `NullReferenceException` la execuție. Prin urmare, ori de câte ori există posibilitatea că un delegat să nu fi fost inițializat, este necesară verificarea înainte de invocare.

Există două forme echivalente:

```csharp
// Forma clasica — verificare explicita
if (notificator != null)
    notificator("CMD-001", "Mesaj test.");

// Forma moderna — operatorul null-conditional
notificator?.Invoke("CMD-001", "Mesaj test.");
```

Forma `?.Invoke(...)` utilizează operatorul null-conditional introdus în C# 6. Dacă delegatul este `null`, expresia nu face nimic și nu aruncă excepție. Dacă delegatul are o valoare, `Invoke` este apelat cu argumentele specificate.

Cele două forme sunt semantic echivalente, dar forma cu `?.Invoke` este preferată în cod modern dintr-un motiv suplimentar: în scenarii multi-threaded, variabila ar putea deveni `null` între momentul verificării `if` și momentul apelului efectiv. Operatorul `?.` evaluează referința o singură dată, eliminând această fereastră de race condition.

### Delegatul ca tip de referință

Un delegat este un tip de referință. Variabila stochează o referință la obiectul delegat, nu obiectul în sine. Această proprietate produce un comportament care merită înțeles explicit.

Când atribui o variabilă delegat alteia, ambele referă același obiect delegat. Operatorul `+=`, însă, nu modifică obiectul existent, ci creează un obiect nou și îl atribuie variabilei. Delegații sunt **imuabili** la nivel de instanță:

```csharp
NotificareClient a = NotificareEmail;
NotificareClient b = a;  // a si b refera acelasi obiect delegat

b += NotificareSMS;
// b refera acum un obiect NOU care contine [NotificareEmail, NotificareSMS]
// a refera in continuare obiectul initial, care contine doar [NotificareEmail]

a("CMD-001", "Test");  // Apeleaza DOAR NotificareEmail
b("CMD-001", "Test");  // Apeleaza NotificareEmail SI NotificareSMS
```

Implicația practică este că operatorul `+=` aplicat unei variabile nu afectează alte variabile care refereau anterior același obiect delegat.


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-3-delegati-actiuni-si-evenimente/delegati.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
