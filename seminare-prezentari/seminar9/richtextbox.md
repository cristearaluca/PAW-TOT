# RichTextBox

`RichTextBox` este o versiune extinsă a `TextBox` care suportă text formatat — fonturi diferite, culori, bold, italic — și gestionează fără probleme volume mari de text. Spre deosebire de `TextBox` cu `Multiline = true`, `RichTextBox` este mai potrivit atunci când conținutul poate fi lung sau când vrei control fin asupra aspectului textului.

În exercițiu îl folosim în două locuri cu roluri diferite: în `Form1` ca zonă de previzualizare read-only a intrării selectate, și în `FormIntrare` ca câmp de editare pentru conținut.

### Proprietăți esențiale

```csharp
// Continutul ca string simplu
rtbContinut.Text = intrare.Continut;

// Citirea continutului
string continut = rtbContinut.Text;

// Modul doar-citire — utilizatorul nu poate edita, dar poate selecta si copia
rtbContinut.ReadOnly = true;

// Bara de defilare verticala — utila pentru texte lungi
rtbContinut.ScrollBars = RichTextBoxScrollBars.Vertical;

// Culoare de fundal — conventia WinForms pentru campuri read-only
rtbContinut.BackColor = SystemColors.Control;
```

`ReadOnly = true` împiedică editarea, dar textul rămâne selectabil și copiabil — comportamentul corect pentru o zonă de previzualizare.

`BackColor = SystemColors.Control` folosește exact culoarea de fundal a ferestrei, semnalând vizual că zona nu este editabilă — aceeași convenție pe care o folosesc `Label`-urile și alte controale read-only din Windows.

### Afișarea condiționată pentru intrările private

La selectarea unei intrări în `ListView`, afișezi conținutul în `rtbContinut`. Intrările marcate ca private nu trebuie să afișeze conținutul real:

```csharp
private void lvIntrari_SelectedIndexChanged(object sender, EventArgs e)
{
    if (lvIntrari.SelectedItems.Count == 0)
    {
        rtbContinut.Text = string.Empty;
        return;
    }

    IntrareDjurnal intrare = lvIntrari.SelectedItems[0].Tag as IntrareDjurnal;

    if (intrare.EstePrivata)
    {
        rtbContinut.ForeColor = Color.Gray;
        rtbContinut.Text = "*** Intrare privată ***";
    }
    else
    {
        rtbContinut.ForeColor = SystemColors.ControlText;
        rtbContinut.Text = intrare.Continut;
    }
}
```

### `RichTextBox` în `FormIntrare` — câmp de editare

În formularul de adăugare/editare, `RichTextBox` este câmpul în care utilizatorul scrie conținutul intrării. Nu este read-only — proprietățile implicite sunt suficiente:

```csharp
// La incarcarea formularului in modul editare
rtbContinut.Text = intrare.Continut;

// La citirea continutului la confirmarea cu OK
string continut = rtbContinut.Text.Trim();
if (string.IsNullOrEmpty(continut))
{
    errorProvider1.SetError(rtbContinut, "Conținutul nu poate fi gol.");
    return;
}
```

`RichTextBox` funcționează cu `ErrorProvider` la fel ca `TextBox` — `SetError` afișează iconița de eroare lângă control.

### Diferența față de `TextBox Multiline`

Ambele pot afișa text pe mai multe rânduri. Diferența principală în contextul exercițiului este că `RichTextBox` gestionează mai bine texte lungi și are o scrollbar mai fluentă. Pentru conținut de jurnal — care poate fi oricât de lung — `RichTextBox` este alegerea mai potrivită.


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-9-serializarea-si-deserializarea/richtextbox.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
