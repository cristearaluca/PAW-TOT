# Exercițiu

Construiești o aplicație desktop pentru gestionarea unui jurnal personal. Fereastra principală afișează toate intrările într-un `ListView`, iar selectarea unei intrări afișează conținutul complet într-un `RichTextBox` alăturat. Colecția poate fi salvată într-un fișier binar și reîncărcată ulterior — fără librării externe, exclusiv cu ce oferă .NET Framework.

### Modelul de date

Declară enumerația `Dispozitie` cu valorile: `Fericit`, `Neutru`, `Trist`, `Entuziasmat`.

Declară clasa `IntrareJurnal` cu proprietățile:

* `DateTime Data`
* `string Titlu`
* `string Continut`
* `Dispozitie Dispozitie`
* `bool EstePrivata`

Marchează clasa cu atributul `[Serializable]` și adaugă un constructor fără parametri.

### Fereastra principală&#x20;

Pe `Form1` adaugă:

* Un `MenuStrip` cu meniurile:
  * **Fișier:** Deschide binar, Salvează binar, Deschide XML, Salvează XML
  * **Intrare:** Adaugă, Editează, Șterge
* Un `ListView` (`lvIntrari`) în jumătatea stângă, configurat în modul `Details` cu coloanele: `Data`, `Titlu`, `Dispoziție`
* Un `RichTextBox` (`rtbContinut`) în jumătatea dreaptă cu `ReadOnly = true`
* Un `Label` de status (`lblStatus`) în partea de jos

Declară în `FakeDatabase` o `List<IntrareJurnal> intrari` inițializată cu o listă goală.

Implementează `RefreshLista()` care golește și repopulează `lvIntrari` din `intrari`. Stochează referința la obiect în `Tag`-ul fiecărui `ListViewItem`.

### Formularul secundar `FormIntrare`

Pe `FormIntrare` adaugă:

* `DateTimePicker` pentru dată (`dtpData`)
* `TextBox` pentru titlu (`txtTitlu`)
* `RichTextBox` pentru conținut (`rtbContinut`)
* `ComboBox` pentru dispoziție (`cmbDispozitie`) — populat din enum cu `Enum.GetValues`
* `CheckBox` pentru privat (`chkEstePrivata`)
* Butoane `btnOk` și `btnAnuleaza` cu `ErrorProvider`

`FormIntrare` primește în constructor un `IntrareDjurnal` — `null` pentru adăugare, obiectul existent pentru editare. Expune rezultatul prin proprietatea publică `IntrareDjurnal Rezultat`.

La click pe `btnOk`, validează că `Titlu` nu este gol și că `Continut` nu este gol, apoi populează `Rezultat` și setează `DialogResult = OK`.

### Operațiile CRUD

Implementează adăugarea, editarea și ștergerea prin `FormIntrare`. La fiecare operație, actualizează `intrari` și apelează `RefreshLista()`.

La selectarea unei intrări în `lvIntrari`, afișează conținutul în `rtbContinut`. Dacă `EstePrivata = true`, afișează `"*** Intrare privată ***"` în loc de conținut.

### Salvarea binară cu `SaveFileDialog`

La click pe **Salvează binar**:

* Afișează un `SaveFileDialog` cu filtrul `"Jurnal binar (*.jrn)|*.jrn"`
* Dacă utilizatorul confirmă, serializează `intrari` cu `BinaryFormatter` în fișierul ales
* Actualizează `lblStatus` cu un mesaj de confirmare

### Încărcarea binară cu `OpenFileDialog`

La click pe **Deschide binar**:

* Afișează un `OpenFileDialog` cu filtrul `"Jurnal binar (*.jrn)|*.jrn"`
* Dacă utilizatorul selectează un fișier, deserializează cu `BinaryFormatter` și înlocuiește `intrari`
* Apelează `RefreshLista()` și actualizează `lblStatus`

### Salvarea XML cu `SaveFileDialog`

Adaugă în meniul **Fișier** itemurile **Salvează XML** și **Deschide XML**.

La click pe **Salvează XML**:

* Afișează un `SaveFileDialog` cu filtrul `"Jurnal XML (*.xml)|*.xml"`
* Dacă utilizatorul confirmă, serializează `intrari` cu `XmlSerializer` folosind un `StreamWriter`
* Actualizează `lblStatus`

Pentru ca `XmlSerializer` să poată serializa lista, declară o clasă wrapper:

```csharp
[Serializable]
public class JurnalWrapper
{
    public List<IntrareJurnal> Intrari { get; set; }
    public JurnalWrapper() { Intrari = new List<IntrareJurnal>(); }
}
```

### Încărcarea XML cu `OpenFileDialog`

La click pe **Deschide XML**:

* Afișează un `OpenFileDialog` cu filtrul `"Jurnal XML (*.xml)|*.xml"`
* Dacă utilizatorul selectează un fișier, deserializează cu `XmlSerializer` și înlocuiește `intrari`
* Apelează `RefreshLista()` și actualizează `lblStatus`

### Exemplu de date

```csharp
public static class FakeDatabase
{
    public static List<IntrareJurnal> intrari = new List<IntrareJurnal>
    {
        new IntrareJurnal
        {
            Data        = new DateTime(2025, 3, 10),
            Titlu       = "Prima zi de primăvară",
            Continut    = "Azi a ieșit soarele pentru prima dată după luni întregi de gri. " +
                          "Am ieșit la plimbare prin parc și am simțit că totul poate fi bine.",
            Dispozitie  = Dispozitie.Fericit,
            EstePrivata = false
        },
        new IntrareJurnal
        {
            Data        = new DateTime(2025, 4, 22),
            Titlu       = "O zi obișnuită",
            Continut    = "Nimic special azi. Am lucrat, am mâncat, am citit puțin înainte de culcare. " +
                          "Uneori normalul e de ajuns.",
            Dispozitie  = Dispozitie.Neutru,
            EstePrivata = false
        },
        new IntrareJurnal
        {
            Data        = new DateTime(2025, 5, 5),
            Titlu       = "Vești proaste",
            Continut    = "Am aflat că proiectul pe care l-am muncit luni întregi a fost anulat. " +
                          "Greu de acceptat. Am nevoie de timp să procesez.",
            Dispozitie  = Dispozitie.Trist,
            EstePrivata = false
        },
        new IntrareJurnal
        {
            Data        = new DateTime(2025, 6, 18),
            Titlu       = "Gânduri personale",
            Continut    = "Am luat o decizie importantă legată de viitorul meu. Nu sunt pregătit " +
                          "să o împărtășesc cu nimeni deocamdată.",
            Dispozitie  = Dispozitie.Entuziasmat,
            EstePrivata = true
        },
        new IntrareJurnal
        {
            Data        = new DateTime(2025, 7, 2),
            Titlu       = "Vacanță meritată",
            Continut    = "Prima zi la munte. Aerul curat, liniștea, și o carte bună. " +
                          "Exact ce aveam nevoie după ultimele luni agitate.",
            Dispozitie  = Dispozitie.Fericit,
            EstePrivata = false
        }
    };
}
```


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-9-serializarea-si-deserializarea/exercitiu.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
