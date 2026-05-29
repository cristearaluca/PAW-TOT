# Exercițiu

Construiește o aplicație desktop cu o singură fereastră care permite gestionarea unui registru de contacte. Datele sunt stocate în memorie pe durata sesiunii. Aplicația demonstrează cele mai importante controale WinForms și modul în care interfața grafică se conectează la logica din cod.

### Structura proiectului și formularul principal

Creează un nou proiect de tip **Windows Forms App (.NET Framework)** în Visual Studio.

Setează pe formularul principal (`Form1`) următoarele proprietăți:

* `Text` = `"Registru Contacte"`
* `Size` = `600, 500`
* `StartPosition` = `CenterScreen`
* `FormBorderStyle` = `FixedSingle`
* `MaximizeBox` = `False`

Adaugă în proiect clasa `Contact` cu proprietățile:

* `string Nume`
* `string Prenume`
* `string Telefon`
* `string Email`
* `bool NotificariActive`

Suprascrie `ToString()` astfel încât să returneze `"Prenume Nume — Telefon"`.

### Formularul de adăugare

Adaugă un `GroupBox` cu textul `"Date contact"` în partea stângă a ferestrei.

În interiorul `GroupBox`-ului, adaugă:

* `Label` + `TextBox` pentru Nume (`txtNume`)
* `Label` + `TextBox` pentru Prenume (`txtPrenume`)
* `Label` + `TextBox` pentru Telefon (`txtTelefon`)
* `Label` + `TextBox` pentru Email (`txtEmail`)
* `CheckBox` cu textul `"Notificări active"` (`chkNotificari`)
* `Button` cu textul `"Adaugă contact"` (`btnAdauga`)

La click pe `btnAdauga`:

* Validează că Nume, Prenume și Telefon nu sunt goale
* Dacă validarea eșuează, afișează un `MessageBox` cu mesajul corespunzător
* Dacă validarea reușește, creează un obiect `Contact`, adaugă-l în lista din memorie și golește câmpurile

### Lista de contacte

Adaugă un `ListBox` (`lstContacte`) în partea dreaptă a ferestrei.

Adaugă deasupra lui un `Label` cu textul `"Contacte"`.

După adăugarea unui contact, actualizează `ListBox`-ul apelând o metodă separată `RefreshLista()` care:

* Golește `lstContacte.Items`
* Parcurge lista de contacte din memorie
* Adaugă fiecare contact cu `Items.Add`

### Căutare în timp real

Adaugă un `TextBox` de căutare (`txtCautare`) deasupra `ListBox`-ului, cu proprietatea `PlaceholderText` = `"Caută după nume..."`.

Abonează-te la evenimentul `TextChanged` al `txtCautare`.

În handler, filtrează lista de contacte din memorie și actualizează `ListBox`-ul să afișeze doar contactele al căror Nume sau Prenume conține textul introdus (comparație case-insensitive).

Dacă `txtCautare` este gol, afișează toate contactele.

### Ștergere cu confirmare

Adaugă un `Button` cu textul `"Șterge contact"` (`btnSterge`) sub `ListBox`.

La click pe `btnSterge`:

* Dacă niciun contact nu este selectat în `ListBox`, afișează `MessageBox` cu avertisment
* Dacă un contact este selectat, afișează un `MessageBox` cu `MessageBoxButtons.YesNo` care cere confirmare
* Dacă utilizatorul confirmă, șterge contactul din lista din memorie și actualizează `ListBox`-ul

### Label de status

Adaugă un `Label` (`lblStatus`) în partea de jos a ferestrei.

Actualizează textul lui `lblStatus` după fiecare operație:

* După adăugare: `"Contact adăugat. Total: N contacte."`
* După ștergere: `"Contact șters. Total: N contacte."`
* La căutare activă: `"Se afișează N din M contacte."`
* Când câmpul de căutare este golit: revine la totalul complet


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-4-introducere-in-windows-forms/exercitiu.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
