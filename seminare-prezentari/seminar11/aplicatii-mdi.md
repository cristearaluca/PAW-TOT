# Aplicații MDI

**MDI** (Multiple Document Interface) este un model de interfață grafică în care o fereastră principală — **containerul MDI** — găzduiește în interior una sau mai multe ferestre secundare numite **ferestre copil**. Ferestrele copil nu sunt ferestre independente ale sistemului de operare: nu au propria intrare în taskbar, nu pot ieși din limitele containerului și sunt gestionate împreună cu el.

Modelul alternativ, **SDI** (Single Document Interface), are o fereastră per document — fiecare independentă, cu propria intrare în taskbar. Aplicații precum Notepad sunt SDI. Aplicații precum versiunile vechi de Microsoft Word sau Excel sunt MDI.

Avantajul MDI pentru aplicații de gestiune a datelor este că toate funcționalitățile sunt organizate într-un singur spațiu de lucru. Utilizatorul nu pierde ferestrele printre alte aplicații deschise și containerul poate oferi funcționalități comune tuturor ferestrelor copil — un meniu global, o bară de status, un meniu Window cu lista ferestrelor deschise.

### Configurarea containerului MDI

Orice `Form` poate deveni container MDI prin setarea proprietății `IsMdiContainer = true`. Aceasta se face din fereastra Properties în Designer sau din cod în constructor:

```csharp
public partial class FormPrincipal : Form
{
    public FormPrincipal()
    {
        InitializeComponent();
        this.IsMdiContainer = true;  // sau setat direct in Designer
    }
}
```

> 📸 **SCREENSHOT:** Fereastra Properties a `FormPrincipal` cu proprietatea `IsMdiContainer` evidențiată și setată pe `True`. Arată și cum arată formularul în Designer după activare — fundalul interior devine gri închis, specific containerelor MDI.

Vizual, un container MDI are fundalul interior distinct față de un formular obișnuit — de obicei gri închis. Ferestrele copil se deschid în interiorul acestei zone.

### Deschiderea ferestrelor copil

O fereastră copil MDI este un `Form` obișnuit, fără nicio configurare specială. Singura diferență față de un formular deschis cu `Show()` obișnuit este că îi setezi proprietatea `MdiParent` înainte de a o afișa:

```csharp
private void mniListaTaskuri_Click(object sender, EventArgs e)
{
    FormLista f = new FormLista();
    f.MdiParent = this;  // this = FormPrincipal, containerul MDI
    f.Show();
}
```

`f.MdiParent = this` înregistrează fereastra copil în containerul MDI. Fără această linie, `f.Show()` deschide o fereastră independentă normală, în afara containerului.

> 📸 **SCREENSHOT:** Aplicația în execuție cu `FormPrincipal` ca container MDI și `FormLista` deschis în interior. Arată că fereastra copil este limitată la spațiul interior al containerului și are propria bară de titlu redusă.

### Verificarea dacă o fereastră copil este deja deschisă

Dacă utilizatorul apasă „Listă task-uri" de mai multe ori, nu vrem să deschidem instanțe multiple ale aceleiași ferestre — aceasta ar crea confuzie și date duplicate. Containerul MDI expune toate ferestrele copil deschise curent prin colecția `MdiChildren`:

```csharp
private void mniListaTaskuri_Click(object sender, EventArgs e)
{
    // Cautam o fereastra de tip FormLista deja deschisa
    foreach (Form f in this.MdiChildren)
    {
        if (f is FormLista)
        {
            f.Activate();  // aducem fereastra existenta in prim-plan
            return;
        }
    }

    // Nu exista — o deschidem
    FormLista lista = new FormLista();
    lista.MdiParent = this;
    lista.Show();
}
```

`Activate()` aduce fereastra în prim-plan și îi dă focus — exact comportamentul pe care îl așteptăm când fereastra există deja.

### `LayoutMdi` — aranjarea ferestrelor copil

Containerul MDI poate aranja automat ferestrele copil prin metoda `LayoutMdi`:

```csharp
this.LayoutMdi(MdiLayout.Cascade);      // cascada — suprapuse decalat
this.LayoutMdi(MdiLayout.TileHorizontal); // alaturate orizontal
this.LayoutMdi(MdiLayout.TileVertical);   // alaturate vertical
this.LayoutMdi(MdiLayout.ArrangeIcons);  // aranjeza iconitele minimizate
```

Nu este necesar în exercițiu, dar este un comportament standard pe care îl găsești în meniurile „Window" ale aplicațiilor MDI clasice.

### `MenuStrip` pe containerul MDI

`MenuStrip`-ul adăugat pe containerul MDI este vizibil permanent, indiferent de ce ferestre copil sunt deschise. Aceasta este una dintre valorile arhitecturii MDI: un singur meniu global controlează toate ferestrele.

Un `MenuStrip` adăugat pe o fereastră copil nu înlocuiește cel al containerului — în WinForms, meniurile ferestrelor copil se pot fuziona cu meniul containerului prin `MergeAction` și `MergeIndex`. Pentru exercițiu nu implementăm fuzionarea — meniul principal rămâne exclusiv pe container.


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-11-aplicatii-mdi/aplicatii-mdi.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
