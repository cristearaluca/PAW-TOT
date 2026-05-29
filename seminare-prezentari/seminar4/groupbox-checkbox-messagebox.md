# GroupBox, CheckBox, MessageBox

### `GroupBox`

`GroupBox` este un container care grupează controale înrudite. Afișează un chenar cu un titlu și comunică utilizatorului că elementele din interior formează o unitate logică. Nu are o funcție de procesare a datelor. Rolul său este exclusiv de organizare vizuală și de structurare a interfeței.

#### Proprietăți esențiale

```csharp
grpDateContact.Text = "Date contact";  // titlul afisat pe chenar
grpDateContact.Enabled = false;        // dezactiveaza toate controalele din interior
```

Proprietatea `Enabled` pe un `GroupBox` are un efect de grup: când o setezi pe `false`, toate controalele din interiorul lui devin automat dezactivate, fără a fi nevoie să le dezactivezi individual. Aceasta este o modalitate convenabilă de a bloca o secțiune întreagă a formularului în timp ce se procesează o operație.

#### Adăugarea controalelor în `GroupBox`

În Designer, tragi controalele direct în interiorul `GroupBox`-ului. Dacă plasezi un control în afara lui și vrei să-l muți înăuntru, tai cu `Ctrl+X` și lipești cu `Ctrl+V` cu `GroupBox`-ul selectat.

Un semn că un control aparține `GroupBox`-ului: dacă muți `GroupBox`-ul pe formular, controalele din interior se mută odată cu el. Dacă un control rămâne pe loc la mutarea `GroupBox`-ului, înseamnă că nu a fost adăugat corect în container.

### `CheckBox`

`CheckBox` reprezintă o opțiune binară: bifat sau nebifat. Utilizatorul poate schimba starea dând click pe el. Este controlul potrivit pentru orice setare care poate fi activată sau dezactivată independent de alte setări.

#### Proprietăți esențiale

```csharp
chkNotificari.Text = "Notificari active";  // textul afisat langa casuta
chkNotificari.Checked = true;              // bifat
chkNotificari.Checked = false;             // nebifat
```

#### Citirea stării în cod

Starea curentă a unui `CheckBox` se citește prin proprietatea `Checked`, care returnează `bool`. Aceasta este valoarea pe care o transmiți obiectului de date la construcție:

```csharp
Contact c = new Contact
{
    Nume = txtNume.Text.Trim(),
    Prenume = txtPrenume.Text.Trim(),
    Telefon = txtTelefon.Text.Trim(),
    Email = txtEmail.Text.Trim(),
    NotificariActive = chkNotificari.Checked
};
```

#### Evenimentul `CheckedChanged`

Se declanșează când utilizatorul bifează sau debifează controlul. Util când vrei să reacționezi imediat la schimbarea stării, de exemplu pentru a activa sau dezactiva alte controale din formular:

```csharp
private void chkNotificari_CheckedChanged(object sender, EventArgs e)
{
    if (chkNotificari.Checked)
        lblStatus.Text = "Notificarile vor fi activate pentru acest contact.";
    else
        lblStatus.Text = "Notificarile sunt dezactivate.";
}
```

Dacă nu ai nevoie să reacționezi în timp real la schimbarea stării, nu este obligatoriu să abonezi acest eveniment. Poți citi `Checked` direct când ai nevoie de valoare, de exemplu la click pe butonul de adăugare.

### `MessageBox`

`MessageBox` este o fereastră de dialog predefinită de sistem care afișează un mesaj și opțional solicită o decizie din partea utilizatorului. Nu se instanțiază cu `new`, ci se apelează static prin `MessageBox.Show(...)`.

Un `MessageBox` deschis **blochează interacțiunea cu restul aplicației** până când utilizatorul îl închide. Acesta este comportamentul dorit: pentru erori de validare și cereri de confirmare, utilizatorul trebuie să răspundă înainte să continue.

#### Forma simplă

```csharp
MessageBox.Show("Contactul a fost adaugat cu succes.");
```

Această formă afișează mesajul cu titlul implicit al aplicației și un singur buton OK.

#### Cu titlu și icon

```csharp
MessageBox.Show(
    "Numele este obligatoriu.",
    "Eroare de validare",
    MessageBoxButtons.OK,
    MessageBoxIcon.Warning
);
```

Parametrii sunt în ordine: mesajul, titlul ferestrei, butoanele afișate, iconul. Toți parametrii după mesaj sunt opționali, dar includerea lor face dialogul mai informativ și mai profesional.

#### Valorile `MessageBoxIcon`

```csharp
MessageBoxIcon.Information   // cerc albastru cu litera i
MessageBoxIcon.Warning       // triunghi galben cu semnul !
MessageBoxIcon.Error         // cerc rosu cu semnul X
MessageBoxIcon.Question      // cerc albastru cu semnul intrebarii
```

Alegerea iconului potrivit este importantă pentru experiența utilizatorului: `Warning` pentru validări care nu blochează complet operația, `Error` pentru probleme care o blochează, `Question` pentru cereri de confirmare, `Information` pentru mesaje neutre.

#### Cu butoane multiple si `DialogResult`

Când afișezi un `MessageBox` cu mai mult de un buton, trebuie să captezi răspunsul utilizatorului. Valoarea returnată de `MessageBox.Show` este de tip `DialogResult`:

```csharp
DialogResult rezultat = MessageBox.Show(
    "Esti sigur ca vrei sa stergi contactul \"" + contact.Prenume + " " + contact.Nume + "\"?",
    "Confirmare stergere",
    MessageBoxButtons.YesNo,
    MessageBoxIcon.Question
);

if (rezultat == DialogResult.Yes)
{
    contacte.Remove(contact);
    RefreshLista();
    lblStatus.Text = "Contact sters. Total: " + contacte.Count + " contacte.";
}
// Daca rezultat == DialogResult.No, nu facem nimic, utilizatorul a anulat
```

#### Valorile `MessageBoxButtons`

```csharp
MessageBoxButtons.OK              // un singur buton OK
MessageBoxButtons.OKCancel        // OK si Cancel
MessageBoxButtons.YesNo           // Yes si No
MessageBoxButtons.YesNoCancel     // Yes, No si Cancel
MessageBoxButtons.RetryCancel     // Retry si Cancel
```

#### Valorile `DialogResult`

```csharp
DialogResult.OK
DialogResult.Cancel
DialogResult.Yes
DialogResult.No
DialogResult.Retry
```

Valorile disponibile corespund butoanelor afișate. Dacă utilizatorul apasă `Yes`, rezultatul este `DialogResult.Yes`; dacă apasă `No`, rezultatul este `DialogResult.No`.

### Combinarea celor trei controale in exercitiu

În registrul de contacte, `GroupBox`, `CheckBox` și `MessageBox` lucrează împreună cu roluri complementare. `GroupBox` grupează vizual câmpurile de introducere a datelor. `CheckBox` permite setarea opțiunii `NotificariActive` fără câmpuri text suplimentare. `MessageBox` protejează împotriva acțiunilor greșite: afișează erori de validare și cere confirmare înainte de ștergere.

Un exemplu complet al operației de ștergere cu confirmare:

```csharp
private void btnSterge_Click(object sender, EventArgs e)
{
    StergeContact();
}

private void StergeContact()
{
    if (lstContacte.SelectedItem == null)
    {
        MessageBox.Show("Selecteaza mai intai un contact din lista.", "Atentie",
            MessageBoxButtons.OK, MessageBoxIcon.Information);
        return;
    }

    Contact selectat = lstContacte.SelectedItem as Contact;

    DialogResult confirmare = MessageBox.Show(
        "Stergi contactul \"" + selectat.Prenume + " " + selectat.Nume + "\"?",
        "Confirmare",
        MessageBoxButtons.YesNo,
        MessageBoxIcon.Question);

    if (confirmare == DialogResult.Yes)
    {
        contacte.Remove(selectat);
        RefreshLista();
        lblStatus.Text = "Contact sters. Total: " + contacte.Count + " contacte.";
    }
}
```

Observați că handler-ul `btnSterge_Click` nu conține nicio logică propriu-zisă, ci apelează metoda `StergeContact()`. Aceasta este separarea de responsabilitate tratată în detaliu în capitolul următor.


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-4-introducere-in-windows-forms/groupbox-checkbox-messagebox.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
