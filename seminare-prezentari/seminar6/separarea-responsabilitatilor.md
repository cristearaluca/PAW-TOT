# Separarea responsabilităților

În laboratorul anterior am construit o aplicație funcțională — angajații se adăugau, se editau, se ștergeau. Dar dacă ne uităm atent la `Form1`, observăm că face prea multe lucruri simultan.

`Form1` gestionează aspectul vizual al ferestrei. Reacționează la acțiunile utilizatorului. Ține lista de angajați într-un câmp privat. Decide când se adaugă un element, când se înlocuiește, când se șterge. Primește date de la formularul secundar prin proprietatea `AngajatModificat` și decide ce face cu ele.

```csharp
// Form1 — Lab 04
public partial class Form1 : Form
{
    // Datele traiesc in Form1
    private List<Angajat> _angajati = new List<Angajat>();

    private void btnAdauga_Click(object sender, EventArgs e)
    {
        using (FormAngajat f = new FormAngajat(null))
        {
            if (f.ShowDialog() == DialogResult.OK)
            {
                // Form1 decide ce face cu datele primite
                _angajati.Add(f.AngajatModificat);
                RefreshLista();
            }
        }
    }
}
```

Această abordare funcționează atâta timp cât aplicația este mică și simplă. Problemele apar pe măsură ce adăugăm cerințe noi.

* **Datele sunt captive în UI.** Lista de angajați trăiește în `Form1`. Dacă dorim o a doua fereastră care să afișeze aceiași angajați, aceasta nu poate accesa datele fără să acceseze `Form1` direct, ceea ce creează o dependență urâtă între ferestre.
* **Schimbarea sursei de date înseamnă modificarea UI-ului.** Dacă dorim să salvăm angajații într-un fișier JSON sau o bază de date în loc de memorie, trebuie să modificăm `Form1`. Un formular de interfață nu ar trebui să știe nimic despre cum sau unde sunt stocate datele.
* **Codul este greu de urmărit și de extins.** Când logica de stocare, logica de validare și logica de prezentare sunt amestecate în aceeași clasă, orice modificare riscă să strice altceva.

Principiul fundamental care rezolvă aceste probleme se numește **Separation of Concerns** — fiecare parte a aplicației are o responsabilitate clară și unică și nu intră în treburile celorlalte.

Aplicat la o aplicație de gestiune a datelor, putem identifica trei responsabilități distincte:

* **Stocarea fizică a datelor.** Undeva trebuie să existe datele propriu-zise. Într-o aplicație reală, aceasta este o bază de date, un fișier, un serviciu web. Astăzi, simulăm această sursă printr-o clasă statică `FakeDatabase`.
* **Accesul la date prin operații CRUD.** Cineva trebuie să știe cum să citească, adauge, modifice și șteargă din sursa de date. Aceasta este responsabilitatea repository-ului — un intermediar care expune operațiile CRUD și abstractizează detaliile sursei de date față de restul aplicației.
* **Prezentarea și interacțiunea cu utilizatorul.** Cineva trebuie să știe cum arată fereastra, ce se întâmplă la un click, cum se afișează datele. Aceasta este responsabilitatea formularelor — și **singura** lor responsabilitate.

### Arhitectura n-tier

**N-tier** (n straturi) este un model arhitectural care organizează aplicația în straturi cu responsabilități distincte, unde fiecare strat comunică cu cel de sub el, niciodată invers. Astăzi introducem trei componente distincte — sursa de date, repository-ul și UI-ul — chiar dacă le grupăm în două straturi formale:

```
┌──────────────────────────────────┐
│       Presentation Layer         │
│                                  │
│   Form1, FormCarte               │
│   Butoane, ListView, TextBox     │
│   Ce vede și face utilizatorul   │
└──────────────┬───────────────────┘
               │ foloseste
               │ (nu și invers)
               ▼
┌──────────────────────────────────┐
│        Repository Layer          │
│                                  │
│   CarteRepository                │
│   GetAll, GetById,               │
│   Add, Update, Delete            │
│   Intermediar pentru date        │
└──────────────┬───────────────────┘
               │ accesează
               ▼
┌──────────────────────────────────┐
│          Data Layer              │
│                                  │
│   FakeDatabase                   │
│   List<Carte> Carti (statica)    │
│   Sursa de date simulata         │
└──────────────────────────────────┘
```

Săgețile merg într-o singură direcție: **Presentation folosește Repository, Repository accesează FakeDatabase**. Niciodată invers. `FakeDatabase` nu știe că există formulare. `CarteRepository` nu știe că există butoane. `Form1` nu știe că datele sunt într-o listă statică.

> 📸 **DIAGRAMA:** Desenează pe tablă această structură cu trei dreptunghiuri suprapuse, săgețile cu direcție unică și clasele listate în fiecare nivel. Subliniază că săgețile nu merg și în sens invers — aceasta este regula arhitecturală fundamentală.

### De ce `FakeDatabase` în loc de date direct în repository

Un detaliu important al acestei arhitecturi: **repository-ul nu deține datele**. El este un intermediar, știe cum să opereze pe date, dar datele propriu-zise trăiesc în altă parte.

Aceasta poate părea o distincție academică, dar are o consecință practică imediată: dacă mâine vrem să înlocuim `FakeDatabase` cu o bază de date reală, modificăm **doar** `CarteRepository`. Este de ajuns să schimbăm `FakeDatabase.Carti.Add(c)` cu o instrucțiune SQL. `Form1` și `FormCarte` nu se ating. Nu știu și nu trebuie să știe ce se află în spatele repository-ului.

Dacă datele ar fi trăit în repository, înlocuirea sursei de date ar fi implicat și modificarea repository-ului în esența lui, nu doar a implementării.

### Ce este CRUD

**CRUD** este acronimul pentru cele patru operații fundamentale pe care le facem cu datele în orice aplicație de gestiune, indiferent de domeniu:

| Literă | Operație | Ce înseamnă                           | Metodă în repository      |
| ------ | -------- | ------------------------------------- | ------------------------- |
| **C**  | Create   | Crearea unui element nou              | `Add(Carte c)`            |
| **R**  | Read     | Citirea unuia sau mai multor elemente | `GetAll()`, `GetById(id)` |
| **U**  | Update   | Modificarea unui element existent     | `Update(Carte c)`         |
| **D**  | Delete   | Ștergerea unui element                | `Delete(Guid id)`         |

Aproape orice aplicație de gestiune implementează aceste patru operații. Repository-ul le expune prin metode cu semnături clare și predictibile, iar stratul de prezentare le apelează fără să știe cum sunt implementate intern.

### Ce câștigăm practic

**Înlocuibilitatea sursei de date.** Dacă mâine `FakeDatabase` devine un fișier JSON sau o bază de date SQL Server, modificăm doar `CarteRepository`. Formularele nu se schimbă.

**Date accesibile din orice fereastră.** Deoarece `FakeDatabase.Carti` este o listă statică, orice instanță de `CarteRepository` din orice fereastră accesează aceleași date. Vom vedea în pagina dedicată de ce aceasta rezolvă problema instanțierii multiple, chiar dacă nu este soluția cea mai corectă arhitectural.

**Claritatea codului.** Când citești `Form1`, știi că nu vei găsi logică de stocare acolo. Când citești `CarteRepository`, știi că nu vei găsi cod de UI. Fiecare clasă are un singur scop clar.


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-6-separarea-responsabilitatilor-listview/separarea-responsabilitatilor.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
