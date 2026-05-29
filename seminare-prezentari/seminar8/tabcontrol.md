# TabControl

Există situații în care o fereastră trebuie să afișeze mai multe seturi de informații distincte, dar nu toate simultan. Poți pune totul pe același formular și te lupți cu spațiul, sau poți organiza conținutul în pagini separate navigate prin tab-uri.

`TabControl` rezolvă exact această problemă. Fiecare pagină afișează un set independent de controale — utilizatorul vede o singură pagină la un moment dat și navighează între ele prin click pe etichetele din partea de sus. În exercițiu, cele trei tipuri de grafice ocupă fiecare câte un tab, fără să concureze pe același spațiu.

### Structura internă

`TabControl` conține o colecție de `TabPage`-uri. Fiecare `TabPage` este un container independent care poate conține orice controale, exact ca un `Panel` sau un `GroupBox`. Controalele adăugate pe un `TabPage` sunt vizibile doar când acel tab este activ.

```
TabControl
 ├── TabPage ("Preț")
 │    └── Chart (chartPret)
 ├── TabPage ("Volum")
 │    └── Chart (chartVolum)
 └── TabPage ("Candlestick")
      └── Chart (chartCandlestick)
```

Această structură face `TabControl` diferit de alte containere: el nu aranjează controalele spatial, ci le suprapune pe pagini separate, afișând câte una la un moment dat.

### Adăugarea și configurarea în Designer

`TabControl` se trage din Toolbox și vine implicit cu două `TabPage`-uri. Pentru a adăuga, redenumi sau șterge tab-uri, selectează controlul în Designer, deschide fereastra Properties și caută proprietatea `TabPages`. Click pe butonul `...` de lângă ea deschide un editor de colecție vizual.

> 📸 **SCREENSHOT:** `TabControl` selectat în Designer cu fereastra Properties vizibilă. Arată proprietatea `TabPages` cu butonul `...` evidențiat, și editorul de colecție deschis cu tab-urile listate și butonul „Add" pentru adăugare.

Fiecare `TabPage` are două proprietăți pe care trebuie să le setezi imediat după creare: `Name` este identificatorul din cod (convenție: `tabPret`, `tabVolum`, `tabCandlestick`), iar `Text` este eticheta vizibilă pe tab (`"Preț"`, `"Volum"`, `"Candlestick"`). Cele două sunt complet independente — confundarea lor este o greșeală frecventă.

### Adăugarea controalelor pe un `TabPage`

Pentru a adăuga un control pe un `TabPage` specific, activezi mai întâi acel tab în Designer dând click pe eticheta lui. Orice control tras din Toolbox cât timp tab-ul respectiv este activ devine copilul acelui `TabPage` — nu al `TabControl`-ului și nu al formularului.

Dacă plasezi un control pe formular fără să activezi mai întâi tab-ul dorit, controlul aparține formularului și este vizibil indiferent de tab-ul activ. Aceasta este una dintre cele mai frecvente greșeli la primul contact cu `TabControl`.

Un `Chart` adăugat pe un `TabPage` cu proprietatea `Dock = Fill` va ocupa toată suprafața paginii, lăsând loc doar pentru bara de tab-uri.

> 📸 **SCREENSHOT:** `TabControl` cu tab-ul „Preț" activ în Designer — eticheta apare în prim-plan. Un control `Chart` este plasat în interiorul lui cu `Dock = Fill`, ocupând toată suprafața tab-ului.

### Navigarea din cod

Activarea unui tab din cod se face prin `SelectedIndex` (index 0-based) sau prin `SelectedTab` (referința la obiectul `TabPage`):

```csharp
// Activare dupa index
tabControl.SelectedIndex = 0;

// Activare dupa referinta la TabPage
tabControl.SelectedTab = tabPret;
```

Citirea tab-ului activ curent:

```csharp
int index = tabControl.SelectedIndex;
TabPage tabActiv = tabControl.SelectedTab;
```

### Evenimentul `SelectedIndexChanged`

Se declanșează de fiecare dată când utilizatorul schimbă tab-ul activ. Este locul potrivit pentru a încărca sau actualiza datele din pagina nou activată. Aceasta permite o strategie de **încărcare leneșă** — în loc să încarci toate graficele la pornirea aplicației, le încarci pe fiecare abia când tab-ul corespunzător devine activ:

```csharp
private void tabControl_SelectedIndexChanged(object sender, EventArgs e)
{
    Companie selectata = cmbCompanie.SelectedItem as Companie;
    if (selectata == null) return;

    if (tabControl.SelectedTab == tabPret)
        IncarcaChartPret(selectata);
    else if (tabControl.SelectedTab == tabVolum)
        IncarcaChartVolum(selectata);
    else if (tabControl.SelectedTab == tabCandlestick)
        IncarcaChartCandlestick(selectata);
}
```

Când operațiile de încărcare sunt costisitoare — interogări la baze de date, calcule complexe — acest pattern evită munca inutilă pentru tab-uri pe care utilizatorul nu le vizitează. Pentru exercițiu, unde datele sunt din memorie, diferența de performanță este neglijabilă, dar patternul merită înțeles și aplicat.

### `Dock = Fill` în cascadă

`TabControl` cu `Dock = Fill` pe `Form1` se redimensionează automat cu fereastra. Controalele din interiorul fiecărui `TabPage` care au și ele `Dock = Fill` ocupă toată suprafața tab-ului disponibilă. Configurarea corectă este în cascadă:

```
Form1
  └── TabControl (Dock = Fill)
        └── TabPage
              └── Chart (Dock = Fill)
```

Fiecare nivel umple containerul său. Dacă omiți `Dock = Fill` pe `Chart`, acesta va rămâne la dimensiunea implicită, flotând în colțul din stânga-sus al tab-ului.


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-8-chart-control/tabcontrol.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
