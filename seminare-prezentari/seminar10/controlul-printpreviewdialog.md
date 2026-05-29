# Controlul PrintPreviewDialog

`PrintPreviewDialog` este o fereastră modală predefinită de Windows Forms care afișează previzualizarea documentului înainte de imprimare. Conține un toolbar cu butoane pentru zoom, navigarea între pagini și trimiterea efectivă la imprimantă — toate gata implementate, fără niciun cod suplimentar din partea ta.

Spre deosebire de `PrintPreviewControl` — un control care poate fi integrat într-un formular propriu și personalizat — `PrintPreviewDialog` este o fereastră completă de sine stătătoare, deschisă cu `ShowDialog()`. Este varianta simplă și directă, potrivită atunci când nu ai nevoie să personalizezi fereastra de previzualizare.

### Utilizarea `PrintPreviewDialog`

Configurarea este minimă: creezi o instanță, îi atribui documentul și o deschizi:

```csharp
private void btnPrintPreview_Click(object sender, EventArgs e)
{
    using (PrintPreviewDialog preview = new PrintPreviewDialog())
    {
        preview.Document = printDocument1;
        preview.Width    = 800;
        preview.Height   = 650;
        preview.ShowDialog();
    }
}
```

`preview.Document = printDocument1` leagă dialogul de componenta `PrintDocument`. Fără această linie, dialogul nu știe ce să afișeze și aruncă excepție.

`ShowDialog()` blochează aplicația și deschide fereastra de previzualizare. La apelul lui, `PrintPreviewDialog` declanșează intern `printDocument1.Print()` care la rândul lui declanșează evenimentul `PrintPage` — handler-ul tău este apelat și conținutul este desenat pe suprafața de previzualizare.

`using` asigură că resursele dialogului sunt eliberate după închidere.

> 📸 **SCREENSHOT:** `PrintPreviewDialog` deschis cu catalogul de note vizibil — lista studenților cu materiile indentate și tabelul de medii la final. Arată toolbar-ul din partea de sus cu butoanele de zoom, navigare și imprimare.

### Toolbar-ul integrat

`PrintPreviewDialog` include un toolbar cu funcționalitate completă, gata implementată:

**Butoane de zoom** — 100%, 75%, 50%, 25%, lățimea paginii, pagina întreagă. Utilizatorul poate ajusta zoom-ul pentru a vedea detaliile sau imaginea de ansamblu.

**Navigare pagini** — butoane înainte și înapoi pentru documente cu mai multe pagini. Dacă `e.HasMorePages` rămâne `false`, aceste butoane sunt dezactivate automat.

**Buton de imprimare** — deschide `PrintDialog` standard al Windows și trimite documentul la imprimantă. Tot codul de desenare din `PrintPage` rulează din nou, de data aceasta pe o imprimantă reală.

Nu trebuie să implementezi nimic din acestea — vin automat cu `PrintPreviewDialog`.

### Legătura cu `PrintDocument` — același cod, două contexte

Aceasta este eleganța fundamentală a arhitecturii: `PrintPreviewDialog` și imprimarea efectivă folosesc exact același mecanism. Ambele declanșează `PrintPage` pe `PrintDocument`. Codul tău de desenare rulează identic în ambele cazuri.

```csharp
// Previzualizare — PrintPage este apelat de PrintPreviewDialog
using (PrintPreviewDialog preview = new PrintPreviewDialog())
{
    preview.Document = printDocument1;
    preview.ShowDialog();
}

// Imprimare efectiva — PrintPage este apelat de Print()
printDocument1.Print();
```

Scrii codul de desenare o singură dată și funcționează în ambele contexte. Nu există cod separat pentru „cum arată în previzualizare" și „cum arată pe hârtie".

### Dimensiunile implicite și personalizarea

Dimensiunile implicite ale `PrintPreviewDialog` sunt adesea prea mici pentru a vedea confortabil un document A4. Setarea explicită a `Width` și `Height` îmbunătățește experiența utilizatorului:

```csharp
preview.Width  = 800;
preview.Height = 650;
```

Poți seta și `Text` pentru a personaliza bara de titlu a ferestrei:

```csharp
preview.Text = "Previzualizare catalog note";
```

Aceasta este toată personalizarea disponibilă direct pe `PrintPreviewDialog`. Dacă ai nevoie de mai mult control — de exemplu, să integrezi previzualizarea într-un formular propriu alături de alte controale — folosești `PrintPreviewControl` în loc de `PrintPreviewDialog`.


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-10-printing/controlul-printpreviewdialog.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
