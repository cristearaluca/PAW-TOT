# Label, TextBox, Button

Înainte de a examina fiecare control în parte, există o regulă pe care o aplici de la primul control adăugat pe formular și pe care o respecți pe tot parcursul proiectului: **redenumești fiecare control imediat după ce îl plasezi pe formular**.

Visual Studio generează automat nume ca `button1`, `textBox1`, `label1`. Aceste nume nu comunică nimic despre rolul controlului în aplicație. Codul care referă `textBox3` sau `button2` devine ilizibil după câteva zile, chiar și pentru persoana care l-a scris.

Convenția standard folosește un prefix scurt urmat de un nume descriptiv:

| Prefix | Control  |
| ------ | -------- |
| `btn`  | Button   |
| `txt`  | TextBox  |
| `lbl`  | Label    |
| `lst`  | ListBox  |
| `chk`  | CheckBox |
| `cmb`  | ComboBox |
| `grp`  | GroupBox |

Exemple concrete din exercițiu: `btnAdauga`, `btnSterge`, `txtNume`, `txtPrenume`, `txtCautare`, `lblStatus`, `lstContacte`, `chkNotificari`.

Redenumirea se face din fereastra Properties, câmpul `(Name)`, care apare primul în lista de proprietăți. Este esențial să înțelegi că `(Name)` și `Text` sunt proprietăți diferite: `(Name)` este identificatorul cu care accesezi controlul în cod, `Text` este textul vizibil pe ecran.

### `Label`

`Label` afișează text pe formular. Nu poate fi editat de utilizator și nu captează focusul. Este folosit atât pentru texte statice (denumiri de câmpuri, titluri de secțiuni) cât și pentru texte dinamice care se actualizează în urma acțiunilor utilizatorului.

#### Proprietăți esențiale

```csharp
lblStatus.Text = "Niciun contact.";
lblStatus.AutoSize = true;
lblStatus.Font = new Font("Segoe UI", 9f);
lblStatus.ForeColor = Color.DarkGray;
```

`AutoSize`, care implicit este `true` pentru `Label`, face ca dimensiunea controlului să se ajusteze automat în funcție de textul afișat. Dacă vrei un `Label` cu dimensiune fixă, de exemplu pentru o bară de status cu lățime constantă, setezi `AutoSize = false` și specifici `Size` explicit.

#### Actualizarea în cod

`Label`-ul este actualizat din orice event handler prin atribuirea proprietății `Text`:

```csharp
lblStatus.Text = "Contact adaugat. Total: " + contacte.Count + " contacte.";
```

Actualizarea este instantanee și vizibilă imediat utilizatorului. Nu este nevoie de nicio metodă de refresh sau de repaint explicit.

### `TextBox`

`TextBox` este câmpul de introducere a textului de bază. Utilizatorul poate scrie în el, modifica conținutul și, dacă nu este setat altfel, poate lipi text din clipboard.

#### Proprietăți esențiale

```csharp
txtNume.Text = "";
txtNume.PlaceholderText = "Nume...";
txtNume.MaxLength = 100;
txtNume.ReadOnly = false;
```

`PlaceholderText` afișează un text gri în câmp când acesta este gol, dispărând automat când utilizatorul începe să scrie. Este disponibil în .NET Framework 4.7 și versiuni ulterioare. Proprietatea `MaxLength` limitează numărul de caractere pe care utilizatorul le poate introduce, fără a necesita validare suplimentară.

Proprietatea `Multiline = true` permite introducerea de text pe mai multe rânduri. Câmpurile `ScrollBars` și `WordWrap` devin relevante în acest caz.

#### Citirea și setarea valorii

```csharp
// Citire
string numeCurent = txtNume.Text;

// Setare
txtNume.Text = "Ion Popescu";

// Golire
txtNume.Clear();
```

În practică, citirea textului dintr-un `TextBox` se face aproape întotdeauna cu `.Trim()` aplicat imediat, pentru a elimina spațiile accidentale de la margini:

```csharp
string numeIntroducing = txtNume.Text.Trim();
```

Aceasta este o regulă de igienă: utilizatorii introduc frecvent spații accidentale, iar codul care nu le elimină produce date inconsistente.

#### Evenimentul `TextChanged`

Se declanșează la fiecare modificare a textului, inclusiv la fiecare tastă apăsată. Este evenimentul potrivit pentru funcționalități care răspund în timp real la ce scrie utilizatorul, cum este căutarea incrementală:

```csharp
private void txtCautare_TextChanged(object sender, EventArgs e)
{
    string termen = txtCautare.Text.Trim().ToLower();

    if (string.IsNullOrEmpty(termen))
    {
        RefreshLista();
        return;
    }

    lstContacte.Items.Clear();
    foreach (Contact c in contacte)
    {
        if (c.Nume.ToLower().Contains(termen) || c.Prenume.ToLower().Contains(termen))
            lstContacte.Items.Add(c);
    }
}
```

Observă că filtrarea operează pe lista din memorie (`contacte`) și actualizează `ListBox`-ul, fără a modifica lista. Datele din memorie rămân intacte și se schimbă doar ce se afișează.

### `Button`

`Button` este elementul de acțiune principal. Utilizatorul dă click pe el pentru a declansa o operație.

#### Proprietăți esențiale

```csharp
btnAdauga.Text = "Adaugă contact";
btnAdauga.Enabled = false;
btnAdauga.BackColor = Color.SteelBlue;
btnAdauga.ForeColor = Color.White;
```

Proprietatea `Enabled`, când este `false`, face butonul vizibil dar nefuncțional: utilizatorul îl vede dar nu poate interacționa cu el. Aceasta este modalitatea standard de a comunica că o acțiune nu este disponibilă în contextul curent, fără a ascunde complet butonul.

#### Evenimentul `Click`

Cel mai folosit eveniment al unui buton. Se abonează cel mai simplu dând dublu-click pe buton în Designer, ceea ce generează automat handler-ul:

```csharp
private void btnAdauga_Click(object sender, EventArgs e)
{
    AdaugaContact();
}
```

Un handler de buton bine scris nu conține logică propriu-zisă. El apelează o metodă care conține logica. Această separare este tratată în detaliu în capitolul despre best practices.

#### Parametrul `sender`

Toți event handlerii WinForms primesc `object sender` ca prim parametru. `sender` este controlul care a declanșat evenimentul. Tipul declarat este `object`, dar obiectul real este controlul concret: un `Button`, un `TextBox` etc.

Dacă același handler este abonat la mai multe controale, `sender` îți permite să identifici care anume a generat evenimentul:

```csharp
private void btnActiune_Click(object sender, EventArgs e)
{
    Button btn = sender as Button;
    if (btn == btnAdauga)
        AdaugaContact();
    else if (btn == btnSterge)
        StergeContact();
}
```

Dacă ai nevoie de proprietăți specifice controlului, faci cast:

```csharp
Button btn = (Button)sender;
btn.Enabled = false;
```


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-4-introducere-in-windows-forms/label-textbox-button.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
