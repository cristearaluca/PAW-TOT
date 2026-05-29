# Exercițiu

Construiești o aplicație MDI pentru gestionarea task-urilor. Fereastra principală MDI conține un meniu prin care poți deschide fereastra cu lista de task-uri sau adăuga un task nou. Task-urile sunt afișate într-un `ListView` cu detalii complete. Click dreapta pe un task afișează un `ContextMenuStrip` cu opțiunile „Marchează ca efectuat", „Redeschide" și „Șterge".

### Modelul de date și `FakeDatabase`

Declară enumerația `Prioritate` cu valorile: `Scazuta`, `Medie`, `Inalta`.

Declară clasa `TodoTask` cu proprietățile:

* `int Id` — generat cu contor static în constructor
* `string Titlu`
* `string Descriere`
* `Prioritate Prioritate`
* `DateTime DataCreare`
* `bool Efectuat`

Declară clasa statică `FakeDatabase` cu `static List<TodoTask> Taskuri` populată cu cel puțin 8 task-uri cu date variate — unele marcate ca efectuate, altele nu.

### Fereastra principală MDI

Creează `FormPrincipal` și setează proprietatea `IsMdiContainer = true`.

Adaugă un `MenuStrip` cu structura:

* **Fișier:** Ieșire
* **Task-uri:** Listă task-uri, Task nou

La click pe **Listă task-uri**: verifică dacă `FormLista` este deja deschis — dacă da, îl aduce în prim-plan cu `Activate()`, dacă nu, îl deschide ca fereastră copil MDI.

La click pe **Task nou**: deschide `FormTaskNou` ca fereastră copil MDI.

La click pe **Ieșire**: `Application.Exit()`.

### Fereastra `FormLista`&#x20;

Creează `FormLista` cu un `ListView` (`lvTaskuri`) în modul `Details` cu coloanele: `#`, `Titlu`, `Prioritate`, `Data creării`, `Status`.

Configurează `lvTaskuri`:

* `FullRowSelect = true`
* `MultiSelect = false`
* `GridLines = true`

Implementează `RefreshLista()` care golește și repopulează `lvTaskuri` din `FakeDatabase.Taskuri`. Task-urile efectuate se afișează cu text gri și stilul italic. Stochează obiectul `TodoTask` în `Tag`.

Adaugă un `ContextMenuStrip` (`cmsTaskuri`) cu itemurile:

* `mniMarcheaza` — „Marchează ca efectuat"
* `mniRedeschide` — „Redeschide"
* `mniSterge` — „Șterge"

Asociază `cmsTaskuri` cu `lvTaskuri` prin proprietatea `ContextMenuStrip`.

La deschiderea meniului contextual (evenimentul `Opening`), activează sau dezactivează itemurile în funcție de starea task-ului selectat:

* Dacă task-ul este **neefectuat**: `mniMarcheaza` activ, `mniRedeschide` dezactivat
* Dacă task-ul este **efectuat**: `mniMarcheaza` dezactivat, `mniRedeschide` activ

Implementează handler-ele:

* `mniMarcheaza_Click` — setează `Efectuat = true` și apelează `RefreshLista()`
* `mniRedeschide_Click` — setează `Efectuat = false` și apelează `RefreshLista()`
* `mniSterge_Click` — afișează `MessageBox` de confirmare `YesNo` cu titlul task-ului; dacă confirmat, elimină din `FakeDatabase.Taskuri` și apelează `RefreshLista()`

### Fereastra `FormTaskNou`

Creează `FormTaskNou` cu:

* `TextBox` pentru titlu (`txtTitlu`)
* `RichTextBox` pentru descriere (`rtbDescriere`)
* `ComboBox` pentru prioritate (`cmbPrioritate`) — populat din enum
* `Button` Salvează (`btnSalveaza`) și `Button` Anulează (`btnAnuleaza`)
* `ErrorProvider` pentru validare

La click pe `btnSalveaza`:

* Validează că titlul nu este gol
* Creează un `TodoTask` cu `DataCreare = DateTime.Now` și `Efectuat = false`
* Adaugă în `FakeDatabase.Taskuri`
* Închide fereastra

La click pe `btnAnuleaza`: închide fereastra fără salvare.

### Date de test

```csharp
public static class FakeDatabase
    {
        public static List<TodoTask> Taskuri = new List<TodoTask>
        {
            new TodoTask { Titlu = "Pregătit prezentare laborator",        Descriere = "Slide-uri pentru seminarul de WinForms",              Prioritate = Prioritate.Inalta,  DataCreare = new DateTime(2024, 10, 1),  Efectuat = true  },
            new TodoTask { Titlu = "Corectat teme studenți",               Descriere = "Tema 3 — delegați și evenimente",                    Prioritate = Prioritate.Inalta,  DataCreare = new DateTime(2024, 10, 3),  Efectuat = true  },
            new TodoTask { Titlu = "Actualizat GitBook cu lab05",          Descriere = "Adăugat paginile despre repository pattern",         Prioritate = Prioritate.Medie,   DataCreare = new DateTime(2024, 10, 5),  Efectuat = false },
            new TodoTask { Titlu = "Trimis feedback teze",                 Descriere = "Feedback individual pentru studenții din seria B",   Prioritate = Prioritate.Inalta,  DataCreare = new DateTime(2024, 10, 7),  Efectuat = false },
            new TodoTask { Titlu = "Instalat SQL Server pe lab",           Descriere = "Pregătit mediul pentru seminarul ADO.NET",           Prioritate = Prioritate.Medie,   DataCreare = new DateTime(2024, 10, 8),  Efectuat = true  },
            new TodoTask { Titlu = "Scris exercițiu lab ADO.NET",          Descriere = "Aplicație cu CRUD complet pe baza de date",         Prioritate = Prioritate.Medie,   DataCreare = new DateTime(2024, 10, 10), Efectuat = false },
            new TodoTask { Titlu = "Rezervat sala pentru prezentări",      Descriere = "Sala C205 pentru sesiunea de prezentări proiecte",  Prioritate = Prioritate.Scazuta, DataCreare = new DateTime(2024, 10, 11), Efectuat = false },
            new TodoTask { Titlu = "Publicat subiecte examen parțial",     Descriere = "Subiecte model pe platforma cursului",              Prioritate = Prioritate.Inalta,  DataCreare = new DateTime(2024, 10, 12), Efectuat = false },
            new TodoTask { Titlu = "Actualizat CV pentru conferință",      Descriere = "Adăugat publicațiile din 2024",                     Prioritate = Prioritate.Scazuta, DataCreare = new DateTime(2024, 10, 13), Efectuat = true  },
            new TodoTask { Titlu = "Verificat proiecte de semestru",      Descriere = "Revizuit cerințele pentru proiectele finale",       Prioritate = Prioritate.Medie,   DataCreare = new DateTime(2024, 10, 14), Efectuat = false },
        };
    }
```


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-11-aplicatii-mdi/exercitiu.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
