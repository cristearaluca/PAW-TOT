# FakeDatabase și Repository Layer

### Ce este `FakeDatabase`

`FakeDatabase` este, în exemplul nostru, o clasă statică care simulează o sursă de date externă (o bază de date, un fișier, un serviciu web). Conține o singură colecție statică de cărți și nimic altceva. Nu are logică, nu are metode, nu are validare. Este pur și simplu un container de date accesibil global.

```csharp
static class FakeDatabase
{
    // Lista statica — exista o singura copie in toata aplicatia,
    // indiferent de cate obiecte sau clase o acceseaza
    public static List<Carte> Carti = new List<Carte>
    {
        new Carte { Titlu = "Clean Code", Autor = "Robert Martin",
                    AnAparitie = 2008, Gen = GenCarte.Tehnic, Pret = 89.99m },
        new Carte { Titlu = "Dune", Autor = "Frank Herbert",
                    AnAparitie = 1965, Gen = GenCarte.StiintaFictiune, Pret = 49.99m },
        new Carte { Titlu = "Sapiens", Autor = "Yuval Noah Harari",
                    AnAparitie = 2011, Gen = GenCarte.Biografie, Pret = 65.00m }
    };
}
```

Cuvântul cheie `static` pe clasă înseamnă că nu poate fi instanțiată, ci există automat la pornirea aplicației și trăiește pe toată durata ei. Cuvântul cheie `static` pe proprietatea `Carti` înseamnă că lista aparține clasei, nu unui obiect, și că există o singură copie a ei în memorie.

Această distincție este importantă: nu contează de câte ori și de unde accesezi `FakeDatabase.Carti` — vorbești mereu despre aceeași listă, nu despre copii separate.

### De ce `FakeDatabase` și nu datele direct în repository

Ar putea părea că e mai simplu să pui lista de cărți direct în `CarteRepository`:

```csharp
// Tentant, dar incorect arhitectural
class CarteRepository
{
    private List<Carte> carti = new List<Carte>();  // repository detine datele

    public List<Carte> GetAll() { return new List<Carte>(carti); }
    // ...
}
```

Problema cu această abordare este că amestecă două responsabilități distincte în aceeași clasă: **stocarea datelor** și **accesul la date**. Repository-ul ar juca simultan rolul de sursă de date și de intermediar.

Consecința practică: dacă mai târziu vrei să înlocuiești lista din memorie cu o bază de date reală, trebuie să rescrii `CarteRepository` în totalitate, pentru că logica de acces (metodele `GetAll`, `Add` etc.) și datele sunt împletite.

Separând `FakeDatabase` de `CarteRepository`, fiecare are un singur rol:

* `FakeDatabase` știe **unde** trăiesc datele
* `CarteRepository` știe **cum** să opereze pe ele

La înlocuirea sursei de date, schimbi doar `FakeDatabase` sau modul în care `CarteRepository` o accesează, iar structura metodelor CRUD rămâne identică.

### Structura repository-ului

`CarteRepository` nu deține nicio colecție proprie. Toate operațiile sale accesează `FakeDatabase.Carti` direct:

```csharp
class CarteRepository
{
    // Nu exista niciun camp privat de date — repository-ul nu detine date

    // READ — toate cartile
    public List<Carte> GetAll()
    {
        // Returnam o copie a listei din FakeDatabase, nu lista originala
        return new List<Carte>(FakeDatabase.Carti);
    }

    // READ — o singura carte dupa id
    public Carte GetById(int id)
    {
        return FakeDatabase.Carti.FirstOrDefault(c => c.Id == id);
    }

    // CREATE — adaugam direct in FakeDatabase
    public void Add(Carte c)
    {
        FakeDatabase.Carti.Add(c);
    }

    // UPDATE — gasim cartea cu acelasi Id si o inlocuim
    public void Update(Carte c)
    {
        int index = FakeDatabase.Carti.FindIndex(x => x.Id == c.Id);
        if (index >= 0)
            FakeDatabase.Carti[index] = c;
    }

    // DELETE — eliminam cartea cu id-ul dat
    public void Delete(int id)
    {
        FakeDatabase.Carti.RemoveAll(c => c.Id == id);
    }
}
```

Toate operațiile vorbesc explicit cu `FakeDatabase.Carti`. Nimeni care citește `CarteRepository` nu poate greși de unde vin datele.

### De ce `GetAll()` returnează o copie

`GetAll()` returnează `new List<Carte>(FakeDatabase.Carti)`, nu `FakeDatabase.Carti` direct. Motivul este același ca și în cazul în care datele ar fi deținute de repository: codul din exterior nu trebuie să poată modifica colecția sursă ocolind repository-ul.

```csharp
// Daca GetAll() ar returna FakeDatabase.Carti direct:
List<Carte> lista = repository.GetAll();
lista.Clear();  // golestem accidental FakeDatabase.Carti!
                // toti cei care acceseza FakeDatabase dupa aceasta nu vor mai gasi nimic
```

Returnând o copie, modificările la lista primită nu afectează `FakeDatabase.Carti`:

```csharp
List<Carte> lista = repository.GetAll();
lista.Clear();  // afecteaza doar copia locala, FakeDatabase.Carti e neatinsa
```

Un detaliu tehnic de reținut: se copiază lista, nu obiectele din ea. Obiectele `Carte` din copie sunt aceleași referințe ca cele din `FakeDatabase.Carti`. Modificarea unei proprietăți pe un obiect primit prin `GetAll()` se va reflecta și în `FakeDatabase`. Aceasta este o limitare acceptabilă pentru exercițiu — în aplicații reale s-ar folosi obiecte imutabile sau clonare profundă.

### Ce nu aparține în repository

Repository-ul nu validează date și nu afișează niciun mesaj vizual. Aceste responsabilități aparțin formularelor din stratul de prezentare.

```csharp
// GRESIT — validare si UI in repository
public void Add(Carte c)
{
    if (string.IsNullOrWhiteSpace(c.Titlu))
    {
        MessageBox.Show("Titlul este obligatoriu.");  // UI in repository — incalca separarea
        return;
    }
    FakeDatabase.Carti.Add(c);
}

// CORECT — repository adauga si atat, validarea e responsabilitatea altcuiva
public void Add(Carte c)
{
    FakeDatabase.Carti.Add(c);
}
```

Dacă repository-ul ar conține `MessageBox.Show`, nu l-am putea reutiliza niciodată într-o aplicație consolă sau web. Menținând repository-ul pur — doar operații pe date — el devine utilizabil în orice context.


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-6-separarea-responsabilitatilor-listview/fakedatabase-si-repository-layer.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
