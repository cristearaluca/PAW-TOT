# Rezumat

## Rezumat — Repository Pattern și `ListView`

***

### 1. Separarea straturilor și CRUD

Problema din Lab 04: lista de angajați trăia în `Form1`, logica de stocare și cea de prezentare erau amestecate. Soluția: trei componente cu responsabilități distincte și dependențe într-o singură direcție.

```
┌──────────────────────────────────┐
│       Presentation Layer         │
│   Form1, FormCarte               │
│   Ce vede și face utilizatorul   │
└──────────────┬───────────────────┘
               │ foloseste (nu si invers)
               ▼
┌──────────────────────────────────┐
│        Repository Layer          │
│   CarteRepository                │
│   Intermediar pentru date        │
└──────────────┬───────────────────┘
               │ acceseaza (nu si invers)
               ▼
┌──────────────────────────────────┐
│          Data Layer              │
│   FakeDatabase                   │
│   List<Carte> Carti (statica)    │
└──────────────────────────────────┘
```

| Concept                  | Ce înseamnă                                                     |
| ------------------------ | --------------------------------------------------------------- |
| **`FakeDatabase`**       | Simulează sursa de date — repository-ul nu deține date          |
| **Repository**           | Intermediar care expune CRUD — nu deține date, nu știe de UI    |
| **Presentation**         | Gestionează interfața — nu știe cum sunt stocate datele         |
| **CRUD**                 | Create, Read, Update, Delete — cele patru operații fundamentale |
| **Direcția dependenței** | UI → Repository → FakeDatabase, niciodată invers                |

### 2. `FakeDatabase` și Repository Layer

```csharp
// Sursa de date — container pur, fara logica
static class FakeDatabase
{
    public static List<Carte> Carti = new List<Carte> { /* date initiale */ };
}

// Repository — intermediar, acceseaza FakeDatabase, nu detine date proprii
class CarteRepository
{
    public List<Carte> GetAll()
        => new List<Carte>(FakeDatabase.Carti);  // copie, nu lista originala

    public Carte GetById(int id)
        => FakeDatabase.Carti.FirstOrDefault(c => c.Id == id);

    public void Add(Carte c)     { FakeDatabase.Carti.Add(c); }

    public void Update(Carte c)
    {
        int i = FakeDatabase.Carti.FindIndex(x => x.Id == c.Id);
        if (i >= 0) FakeDatabase.Carti[i] = c;
    }

    public void Delete(int id)   { FakeDatabase.Carti.RemoveAll(c => c.Id == id); }
}
```

| Concept                         | Ce înseamnă                                                 |
| ------------------------------- | ----------------------------------------------------------- |
| **`static List<Carte> Carti`**  | O singură copie în memorie, accesibilă global               |
| **Repository nu deține date**   | Accesează `FakeDatabase.Carti` — este intermediar, nu sursă |
| **`GetAll()` returnează copie** | Protejează `FakeDatabase.Carti` de modificări accidentale   |
| **Contor static pentru `Id`**   | Fiecare obiect nou primește automat un id unic și imuabil   |
| **Fără validare, fără UI**      | Validarea și mesajele vizuale aparțin prezentării           |

### 3. Cum se conectează UI-ul la Repository — instanțiere și problema ei

Varianta din exercițiu — fiecare formular face `new CarteRepository()` — funcționează în aparență deoarece `FakeDatabase.Carti` este statică: toate instanțele de repository accesează aceeași listă. Dar această corectitudine este fragilă — dacă sursa de date s-ar schimba, problema ar reapărea.

Soluția robustă, indiferent de sursa de date, este transmiterea prin constructor:

| Abordare                         | Cum funcționează                                           | Avantaje           | Dezavantaje                                 |
| -------------------------------- | ---------------------------------------------------------- | ------------------ | ------------------------------------------- |
| **Instanțiere locală**           | `new Repository()` în fiecare formular                     | Simplu             | Corect doar cu surse statice; fragil altfel |
| **Transmitere prin constructor** | `Form1` creează, transmite prin `new FormCarte(repo, id)`  | Explicit, robust   | Lanț lung la multe niveluri                 |
| **Singleton**                    | `Repository.GetInstanta()` returnează mereu același obiect | Accesibil global   | Greu de testat, dependență implicită        |
| **Dependency Injection**         | Container gestionează crearea și livrarea                  | Flexibil, testabil | Complexitate suplimentară                   |

```csharp
// Solutia recomandata — transmitere prin constructor
// Form1: creeaza instanta o singura data
private CarteRepository repository = new CarteRepository();
using (var f = new FormCarte(repository, null))
    if (f.ShowDialog() == DialogResult.OK) RefreshLista();

// FormCarte: primeste instanta, nu o creeaza
public FormCarte(CarteRepository repository, int? id)
{
    this.repository = repository;
    this.id = id;
}
```

### 4. `ListView` în modul `Details`

```csharp
// Configurare
lvCarti.View = View.Details;
lvCarti.FullRowSelect = true;
lvCarti.MultiSelect = false;
lvCarti.GridLines = true;

// Adaugare coloane (in Form_Load, inainte de populare)
lvCarti.Columns.Add("Titlu", 200, HorizontalAlignment.Left);
lvCarti.Columns.Add("Preț", 80, HorizontalAlignment.Right);

// Populare
private void RefreshLista()
{
    lvCarti.Items.Clear();
    foreach (Carte c in repository.GetAll())
    {
        ListViewItem item = new ListViewItem(c.Titlu);
        item.SubItems.Add(c.Autor);
        item.SubItems.Add(c.AnAparitie.ToString());
        item.SubItems.Add(c.Gen.ToString());
        item.SubItems.Add(c.Pret.ToString("F2") + " RON");
        item.Tag = c;   // legatura rand-obiect — esential pentru CRUD
        lvCarti.Items.Add(item);
    }
}

// Recuperare la selectare
Carte selectata = lvCarti.SelectedItems[0].Tag as Carte;
```

| Concept                             | Ce înseamnă                                                                |
| ----------------------------------- | -------------------------------------------------------------------------- |
| **`View = Details`**                | Modul tabel cu rânduri și coloane                                          |
| **`ListViewItem`**                  | Un rând în `ListView`                                                      |
| **`item.SubItems.Add(text)`**       | Textul pentru coloanele 2, 3, 4... — ordinea corespunde ordinii coloanelor |
| **`item.Tag = obiect`**             | Stochează referința la obiectul de date — recuperat la selecție            |
| **`SelectedItems[0].Tag as Carte`** | Recuperează obiectul fără căutări în repository                            |
| **`RefreshLista()`**                | Golește și repopulează manual după orice modificare                        |

| Eveniment              | Când se declanșează                                      |
| ---------------------- | -------------------------------------------------------- |
| `SelectedIndexChanged` | La schimbarea selecției — activează/dezactivează butoane |
| `DoubleClick`          | Shortcut pentru editare direct din listă                 |


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-6-separarea-responsabilitatilor-listview/rezumat.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
