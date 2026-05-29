# Bune practici în Windows Forms

### Separarea logicii de interfata

Cea mai importantă regulă în WinForms, și cea mai frecvent încălcată, este aceasta: **event handlerii nu ar trebui să conțină logică de business**.

Un event handler are un singur rol: să reacționeze la o acțiune a utilizatorului. Logica propriu-zisă ( validarea, modificarea datelor, calculele) aparține unor metode separate pe care handler-ul le apelează. Handler-ul este punctul de intrare, nu locul unde se rezolvă problema.

Diferența devine clară printr-o comparație directă.

**Varianta greșită: logică în handler:**

```csharp
private void btnAdauga_Click(object sender, EventArgs e)
{
    if (string.IsNullOrWhiteSpace(txtNume.Text))
    {
        MessageBox.Show("Numele este obligatoriu.");
        return;
    }
    if (string.IsNullOrWhiteSpace(txtPrenume.Text))
    {
        MessageBox.Show("Prenumele este obligatoriu.");
        return;
    }
    Contact c = new Contact();
    c.Nume = txtNume.Text.Trim();
    c.Prenume = txtPrenume.Text.Trim();
    c.Telefon = txtTelefon.Text.Trim();
    c.NotificariActive = chkNotificari.Checked;
    contacte.Add(c);
    lstContacte.Items.Clear();
    foreach (Contact contact in contacte)
        lstContacte.Items.Add(contact);
    txtNume.Clear();
    txtPrenume.Clear();
    txtTelefon.Clear();
    lblStatus.Text = "Contact adaugat. Total: " + contacte.Count;
}
```

**Varianta corectă: handler apelează metode:**

```csharp
private void btnAdauga_Click(object sender, EventArgs e)
{
    AdaugaContact();
}

private void AdaugaContact()
{
    if (!ValideazaFormular())
        return;

    Contact c = CitesteFormular();
    contacte.Add(c);
    RefreshLista();
    GolesteFormular();
    lblStatus.Text = "Contact adaugat. Total: " + contacte.Count + " contacte.";
}

private bool ValideazaFormular()
{
    if (string.IsNullOrWhiteSpace(txtNume.Text))
    {
        MessageBox.Show("Numele este obligatoriu.", "Eroare",
            MessageBoxButtons.OK, MessageBoxIcon.Warning);
        txtNume.Focus();
        return false;
    }
    if (string.IsNullOrWhiteSpace(txtPrenume.Text))
    {
        MessageBox.Show("Prenumele este obligatoriu.", "Eroare",
            MessageBoxButtons.OK, MessageBoxIcon.Warning);
        txtPrenume.Focus();
        return false;
    }
    return true;
}

private Contact CitesteFormular()
{
    return new Contact
    {
        Nume = txtNume.Text.Trim(),
        Prenume = txtPrenume.Text.Trim(),
        Telefon = txtTelefon.Text.Trim(),
        Email = txtEmail.Text.Trim(),
        NotificariActive = chkNotificari.Checked
    };
}

private void GolesteFormular()
{
    txtNume.Clear();
    txtPrenume.Clear();
    txtTelefon.Clear();
    txtEmail.Clear();
    chkNotificari.Checked = false;
    txtNume.Focus();
}
```

Varianta a doua este mai lungă, dar fiecare metodă face un singur lucru cu un nume clar. `ValideazaFormular` poate fi apelată din orice alt loc care are nevoie de aceeași validare. `CitesteFormular` poate fi reutilizată. `GolesteFormular` poate fi apelată și din alte contexte, de exemplu la un buton de resetare. Codul devine navigabil și modificabil fără teama de a strica altceva.

### Denumirea controalelor

Redenumește fiecare control imediat după ce îl adaugi în Designer. Codul care referă `button1` sau `textBox3` este practic de necitit după câteva zile, chiar și de persoana care l-a scris.

Convenția cu prefix scurt aplicată în exercițiu:

```
btnAdauga       btnSterge
txtNume         txtPrenume      txtTelefon      txtEmail        txtCautare
lblStatus       lblNume         lblPrenume
lstContacte
chkNotificari
grpDateContact
```

Redenumirea se face din fereastra Properties, câmpul `(Name)`. Este esențial să reții că `(Name)` și `Text` sunt proprietăți complet diferite: `(Name)` este identificatorul cu care accesezi controlul în cod, `Text` este ce vede utilizatorul pe ecran. Schimbarea lui `Text` nu schimbă `(Name)` și invers.

Dacă ai abonat deja un event handler înainte de a redenumi controlul, Visual Studio poate redenumi automat și handler-ul dacă folosești Rename (`F2`) pe numele controlului în Properties. Dacă nu, va trebui să actualizezi manual și numele din `Form1.Designer.cs`, ceea ce este un motiv în plus să redenumești controalele înainte de a abona evenimente.

### Ce aparține în Designer față de cod

O separare clară între ce se setează în Designer și ce se setează în cod face formularul mai ușor de înțeles și de menținut.

**În Designer (fereastra Properties):** Proprietăți vizuale statice care nu depind de datele din aplicație: `Text`, `Size`, `Location`, `Font`, `BackColor`, `ForeColor`, `FormBorderStyle`, `StartPosition`. Tot ce arată la fel indiferent de starea aplicației se setează în Designer.

**În cod, în `Form1.cs`:** Orice proprietate care depinde de date sau de starea aplicației.

```csharp
// In Designer - text fix, nu se schimba
// btnAdauga.Text = "Adauga contact"

// In cod - depinde de starea aplicatiei
btnSterge.Enabled = lstContacte.SelectedItem != null;
lblStatus.Text = "Total: " + contacte.Count + " contacte.";
this.Text = "Registru Contacte " + DateTime.Now.Year;
```

### Gestionarea erorilor in UI

Nu arunca excepții în event handlers WinForms. O excepție neprinsă dintr-un handler afișează o fereastră de eroare neplăcută pentru utilizator sau, în unele configurații, închide aplicația. Validarea datelor se face explicit, iar erorile se comunică utilizatorului prin `MessageBox`.

```csharp
// Greșit - exceptie aruncata in handler
private void btnAdauga_Click(object sender, EventArgs e)
{
    if (string.IsNullOrEmpty(txtNume.Text))
        throw new ArgumentException("Numele este obligatoriu");
}

// Corect - validare cu MessageBox si return
private void btnAdauga_Click(object sender, EventArgs e)
{
    if (string.IsNullOrEmpty(txtNume.Text))
    {
        MessageBox.Show("Numele este obligatoriu.", "Eroare",
            MessageBoxButtons.OK, MessageBoxIcon.Warning);
        txtNume.Focus();
        return;
    }
}
```

Principiul din capitolul despre excepții (conform căruia excepțiile sunt pentru situații excepționale, nu pentru flux de control normal) se aplică și mai strict în UI. Validarea datelor introduse de utilizator este o situație normală și așteptată, nu o situație excepțională.

### Focus după erori de validare

Când validarea eșuează și afișezi un `MessageBox` de eroare, mută cursorul în câmpul cu problema folosind `Focus()`. Utilizatorul nu trebuie să dea click manual pe câmpul pe care trebuie să-l corecteze. Experiența devine frustrantă mai ales în formulare cu multe câmpuri.

```csharp
if (string.IsNullOrWhiteSpace(txtTelefon.Text))
{
    MessageBox.Show("Telefonul este obligatoriu.", "Eroare",
        MessageBoxButtons.OK, MessageBoxIcon.Warning);
    txtTelefon.Focus();
    txtTelefon.SelectAll();  // selecteaza tot textul din camp, usor de inlocuit
    return;
}
```

`SelectAll()` aplicat după `Focus()` selectează tot textul din câmp, astfel că utilizatorul poate suprascrie imediat fără să fie nevoie să selecteze manual.

### `Trim` la citirea textului

Utilizatorii introduc frecvent spații accidentale la începutul sau sfârșitul unui câmp de text, în special când copiaza si lipesc valori. Aplică `.Trim()` la orice citire din `TextBox` care produce date ce vor fi stocate sau comparate:

```csharp
// Fara Trim - " Ion " trece validarea IsNullOrWhiteSpace, dar ajunge cu spatii in Contact
string nume = txtNume.Text;

// Cu Trim - spatiile de la margini sunt eliminate
string nume = txtNume.Text.Trim();
```

O subtilitate: `string.IsNullOrWhiteSpace(txtNume.Text)` returnează `true` și pentru un câmp care conține numai spații. Deci validarea cu `IsNullOrWhiteSpace` este mai robustă decât `IsNullOrEmpty` pentru câmpuri text introduse de utilizator.

### Actualizarea UI-ului după operații

Menține un principiu simplu: după orice operație care modifică datele din memorie, actualizezi imediat interfața. Nu lăsa `ListBox`-ul, label-urile sau butoanele în stări inconsistente față de datele reale.

Ordinea este întotdeauna: mai întâi modifici datele, apoi actualizezi UI-ul.

```csharp
private void StergeContact()
{
    Contact selectat = lstContacte.SelectedItem as Contact;
    if (selectat == null) return;

    contacte.Remove(selectat);                       // 1. modifica datele din memorie
    RefreshLista();                                  // 2. actualizeaza ListBox-ul
    lblStatus.Text = "Contact sters. Total: "        // 3. actualizeaza labelul de status
        + contacte.Count + " contacte.";
}
```

Niciodată invers. Dacă actualizezi UI-ul înainte să modifici datele și apare o eroare în modificarea datelor, UI-ul va afișa o stare care nu corespunde realității din memorie.


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-4-introducere-in-windows-forms/bune-practici-in-windows-forms.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
