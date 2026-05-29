# ListBox

`ListBox` afișează o listă verticală de elemente din care utilizatorul poate selecta unul sau mai multe. Este controlul potrivit oriunde ai o colecție de obiecte pe care vrei să le afișezi și să permiți interacțiunea cu ele.

Există o distincție fundamentală pe care trebuie să o înțelegi de la început: **`ListBox`-ul și colecția ta din memorie sunt două lucruri complet separate**. `ListBox`-ul afișează elemente; el nu stochează datele aplicației și nu este sursa de adevăr. Sursa de adevăr este colecția din cod, de exemplu `List<Contact> contacte`. Ori de câte ori colecția se modifică, `ListBox`-ul trebuie actualizat manual pentru a reflecta schimbarea.

Această separare este intenționată și valoroasă. Operațiile de filtrare sau căutare pot modifica ce afișează `ListBox`-ul fără a modifica datele din memorie, exact comportamentul dorit pentru o funcție de căutare.

### Adăugarea și eliminarea elementelor

`ListBox`-ul expune colecția sa de elemente prin proprietatea `Items`, de tip `ListBox.ObjectCollection`. Aceasta acceptă orice obiect de tip `object`.

```csharp
// Adaugare un element
lstContacte.Items.Add(contact);

// Adaugare mai multor elemente dintr-o colectie
lstContacte.Items.AddRange(contacte.ToArray());

// Eliminare un element specific (prin referinta)
lstContacte.Items.Remove(contact);

// Eliminare dupa index
lstContacte.Items.RemoveAt(0);

// Stergere completa a tuturor elementelor
lstContacte.Items.Clear();
```

Când adaugi un obiect, `ListBox`-ul apelează automat `ToString()` pe el pentru a determina textul afișat. Acesta este motivul pentru care suprascriem `ToString()` pe clasele al căror obiect vrem să apară în liste:

```csharp
public override string ToString()
{
    return Prenume + " " + Nume + " - " + Telefon;
}
```

Fără această suprascriere, `ListBox`-ul ar afișa numele complet al tipului, ceva de genul `RegistruContacte.Contact`, care nu este util.

### Patternul `RefreshLista()`

Cel mai clar și mai predictibil mod de a ține `ListBox`-ul sincronizat cu lista din memorie este o metodă dedicată care golește `ListBox`-ul și îl repopulează complet la fiecare apel:

```csharp
private void RefreshLista()
{
    lstContacte.Items.Clear();
    foreach (Contact c in contacte)
        lstContacte.Items.Add(c);
}
```

Această metodă se apelează după orice operație care modifică lista din memorie:

```csharp
contacte.Add(contact);
RefreshLista();

contacte.Remove(contact);
RefreshLista();
```

Alternativa de a adăuga direct în `ListBox` cu `Items.Add` în paralel cu modificarea listei din memorie funcționează pentru cazuri simple, dar devine dificil de gestionat în prezența funcționalităților de căutare sau filtrare. Patternul `RefreshLista()` este mai explicit și mai ușor de urmărit.

### Selectarea elementelor

`ListBox`-ul expune mai multe proprietăți pentru a determina ce element este selectat în momentul curent:

```csharp
// Indexul elementului selectat in ListBox (-1 daca nimic nu e selectat)
int index = lstContacte.SelectedIndex;

// Elementul selectat ca object (null daca nimic nu e selectat)
object elementSelectat = lstContacte.SelectedItem;

// Cast la tipul concret pentru a accesa proprietatile specifice
Contact contactSelectat = lstContacte.SelectedItem as Contact;
```

Verificarea dacă ceva este selectat se face prin `SelectedItem != null` sau prin `SelectedIndex != -1`. Ambele sunt echivalente; `SelectedItem != null` este mai explicit:

```csharp
private void btnSterge_Click(object sender, EventArgs e)
{
    if (lstContacte.SelectedItem == null)
    {
        MessageBox.Show("Selecteaza un contact pentru a-l sterge.", "Atentie",
            MessageBoxButtons.OK, MessageBoxIcon.Information);
        return;
    }

    Contact selectat = lstContacte.SelectedItem as Contact;
    contacte.Remove(selectat);
    RefreshLista();
}
```

O subtilitate importantă: dacă obții indexul selectat cu `SelectedIndex` și folosești acel index pentru a accesa lista din memorie cu `contacte[index]`, trebuie să te asiguri că `ListBox`-ul afișează exact aceleași elemente în aceeași ordine ca lista din memorie. Dacă ai aplicat filtrare, indexul din `ListBox` nu mai corespunde indexului din lista completă. Varianta mai sigură este să lucrezi direct cu referința obiectului prin `SelectedItem as Contact`.

### Evenimentul `SelectedIndexChanged`

Se declanșează de fiecare dată când selecția din `ListBox` se schimbă, fie prin click, fie prin taste. Util pentru a reactiona la selecția utilizatorului: afișarea detaliilor elementului selectat sau activarea și dezactivarea butoanelor în funcție de starea selecției.

```csharp
private void lstContacte_SelectedIndexChanged(object sender, EventArgs e)
{
    btnSterge.Enabled = lstContacte.SelectedItem != null;
}
```

Această abordare este preferabilă față de verificarea selecției în handler-ul butonului: butonul este dezactivat când nu există selecție și activat când există, astfel că utilizatorul primește feedback vizual imediat.

### Selecție multiplă

Implicit, `ListBox`-ul permite selectarea unui singur element. Proprietatea `SelectionMode` controlează acest comportament:

```csharp
lstContacte.SelectionMode = SelectionMode.One;           // un singur element (implicit)
lstContacte.SelectionMode = SelectionMode.MultiSimple;   // mai multe cu click simplu
lstContacte.SelectionMode = SelectionMode.MultiExtended; // mai multe cu Shift si Ctrl
```

Când `SelectionMode` nu este `One`, proprietatea `SelectedItem` returnează doar ultimul element selectat. Pentru accesul la toate elementele selectate, folosești `SelectedItems`:

```csharp
foreach (Contact c in lstContacte.SelectedItems)
{
    contacte.Remove(c);
}
RefreshLista();
```

În exercițiu se lucrează cu `SelectionMode.One`, care este implicit și suficient pentru cerințele aplicației.


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-4-introducere-in-windows-forms/listbox.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
