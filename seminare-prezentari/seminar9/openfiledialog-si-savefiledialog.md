# OpenFileDialog și SaveFileDialog

Când o aplicație trebuie să deschidă sau să salveze un fișier, utilizatorii se așteaptă să navigheze vizual prin sistemul de fișiere — nu să tasteze o cale manual într-un `TextBox`. `OpenFileDialog` și `SaveFileDialog` sunt implementările Windows standard ale acestor dialoguri, identice cu cele din Notepad, Word sau orice altă aplicație. Le configurezi din cod, le afișezi cu `ShowDialog()` și citești calea aleasă de utilizator prin proprietatea `FileName`.

### `OpenFileDialog` — selectarea unui fișier

`OpenFileDialog` permite utilizatorului să navigheze și să selecteze un fișier existent. Structura de bază:

```csharp
using (OpenFileDialog dialog = new OpenFileDialog())
{
    dialog.Title = "Deschide jurnal";
    dialog.Filter = "Jurnal binar (*.jrn)|*.jrn";

    if (dialog.ShowDialog() == DialogResult.OK)
    {
        string cale = dialog.FileName;
        // ... folosesti cale pentru a incarca fisierul
    }
}
```

`ShowDialog()` blochează aplicația până când utilizatorul selectează un fișier sau anulează. Returnează `DialogResult.OK` dacă utilizatorul a confirmat, sau `DialogResult.Cancel` dacă a apăsat Anulează sau a închis cu X. Verificarea rezultatului înainte de a folosi `FileName` este obligatorie — dacă utilizatorul anulează, `FileName` este un string gol.

> 📸 **SCREENSHOT:** `OpenFileDialog` deschis, arătând navigarea prin foldere, filtrul activ în colțul din dreapta jos (`*.jrn`) și butonul „Deschide" evidențiat.

### `SaveFileDialog` — alegerea destinației

`SaveFileDialog` permite utilizatorului să aleagă unde și cu ce nume să salveze un fișier:

```csharp
using (SaveFileDialog dialog = new SaveFileDialog())
{
    dialog.Title = "Salvează jurnal";
    dialog.Filter = "Jurnal binar (*.jrn)|*.jrn";
    dialog.DefaultExt = "jrn";
    dialog.OverwritePrompt = true;

    if (dialog.ShowDialog() == DialogResult.OK)
    {
        string cale = dialog.FileName;
        // ... folosesti cale pentru a salva fisierul
    }
}
```

`DefaultExt` adaugă automat extensia dacă utilizatorul nu o include. Dacă scrie `jurnal_2024` și `DefaultExt = "jrn"`, fișierul se salvează ca `jurnal_2024.jrn`.

`OverwritePrompt = true` afișează un dialog de confirmare dacă utilizatorul alege un fișier care există deja. Fără ea, fișierul existent este suprascris silențios.

> 📸 **SCREENSHOT:** `SaveFileDialog` deschis cu câmpul de nume completat și filtrul `*.jrn` activ. Dacă există un fișier cu același nume, arată și dialogul de confirmare suprascriere.

### Sintaxa pentru filtre

Proprietatea `Filter` controlează ce tipuri de fișiere sunt vizibile. Sintaxa este o înșiruire de perechi `descriere|pattern` separate prin `|`:

```csharp
// Un singur tip
dialog.Filter = "Jurnal binar (*.jrn)|*.jrn";

// Mai multe tipuri
dialog.Filter = "Jurnal binar (*.jrn)|*.jrn|Toate fișierele (*.*)|*.*";
```

Fiecare pereche are două string-uri: descrierea afișată utilizatorului și pattern-ul de filtrare. Când există mai multe tipuri, utilizatorul poate schimba filtrul dintr-un dropdown în colțul din dreapta jos al dialogului.

### `InitialDirectory`

Fără `InitialDirectory`, dialogul se deschide într-un director imprevizibil. Este bună practică să specifici un director relevant:

```csharp
dialog.InitialDirectory = Environment.GetFolderPath(Environment.SpecialFolder.MyDocuments);
```


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-9-serializarea-si-deserializarea/openfiledialog-si-savefiledialog.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
