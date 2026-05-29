# Rezumat

### `MenuStrip`

| Concept                       | Ce înseamnă                                                    |
| ----------------------------- | -------------------------------------------------------------- |
| **`MenuStrip`**               | Bara de meniu clasică — se atașează automat la marginea de sus |
| **`ToolStripMenuItem`**       | Un item de meniu la orice nivel                                |
| **`"Type Here"` în Designer** | Adaugă itemuri direct în Designer                              |
| **Prefixul `mni`**            | Convenție de denumire pentru itemurile de meniu                |
| **Dublu-click în Designer**   | Generează și abonează handler-ul `Click`                       |
| **`ToolStripSeparator`**      | Linie de separare vizuală                                      |
| **`Enabled = false`**         | Dezactivează un item — estompat, inaccesibil                   |

```csharp
// Activarea editare/stergere doar cand e selectat ceva
private void lvIntrari_SelectedIndexChanged(object sender, EventArgs e)
{
    bool esteSelectat = lvIntrari.SelectedItems.Count > 0;
    mniEditeaza.Enabled = esteSelectat;
    mniSterge.Enabled = esteSelectat;
}
```

### `OpenFileDialog` și `SaveFileDialog`

| Concept               | Ce înseamnă                                         |
| --------------------- | --------------------------------------------------- |
| **`ShowDialog()`**    | Afișează dialogul — returnează `OK` sau `Cancel`    |
| **`FileName`**        | Calea completă — citită doar după `DialogResult.OK` |
| **`Filter`**          | Tipurile vizibile — `"Jurnal binar (*.jrn)\|*.jrn"` |
| **`DefaultExt`**      | Extensia adăugată automat                           |
| **`OverwritePrompt`** | Confirmare înainte de suprascriere                  |

```csharp
// Deschidere
using (OpenFileDialog d = new OpenFileDialog())
{
    d.Filter = "Jurnal binar (*.jrn)|*.jrn";
    if (d.ShowDialog() == DialogResult.OK)
        Deschide(d.FileName);
}

// Salvare
using (SaveFileDialog d = new SaveFileDialog())
{
    d.Filter = "Jurnal binar (*.jrn)|*.jrn";
    d.DefaultExt = "jrn";
    d.OverwritePrompt = true;
    if (d.ShowDialog() == DialogResult.OK)
        Salveaza(d.FileName);
}
```

### `RichTextBox`

| Concept                                | Ce înseamnă                                    |
| -------------------------------------- | ---------------------------------------------- |
| **`ReadOnly = true`**                  | Previne editarea — textul rămâne selectabil    |
| **`BackColor = SystemColors.Control`** | Semnalizare vizuală standard pentru read-only  |
| **`ScrollBars = Vertical`**            | Defilare pentru texte lungi                    |
| **`ForeColor`**                        | Culoarea textului — gri pentru intrări private |

```csharp
// Afisare conditionata
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
```

### Serializare binară cu `BinaryFormatter`

```csharp
[Serializable]
public class IntrareDjurnal
{
    public DateTime Data { get; set; }
    public string Titlu { get; set; }
    public string Continut { get; set; }
    public Dispozitie Dispozitie { get; set; }
    public bool EstePrivata { get; set; }
    public IntrareDjurnal() { }  // necesar pentru deserializare
}
```

```csharp
// Salvare binara
BinaryFormatter formatter = new BinaryFormatter();
using (FileStream stream = new FileStream(cale, FileMode.Create))
    formatter.Serialize(stream, intrari);

// Incarcare binara
using (FileStream stream = new FileStream(cale, FileMode.Open))
    intrari = (List<IntrareDjurnal>)formatter.Deserialize(stream);
```

| Concept                        | Ce înseamnă                                 |
| ------------------------------ | ------------------------------------------- |
| **`[Serializable]`**           | Obligatoriu pe clasa serializată            |
| **Constructor fără parametri** | Necesar pentru deserializare                |
| **`FileMode.Create`**          | Creează sau suprascrie — pentru salvare     |
| **`FileMode.Open`**            | Deschide existent — pentru citire           |
| **`using` pe `FileStream`**    | Obligatoriu — altfel fișierul rămâne blocat |

### Serializare XML cu `XmlSerializer`

```csharp
[Serializable]
public class JurnalWrapper
{
    public List<IntrareDjurnal> Intrari { get; set; }
    public JurnalWrapper() { Intrari = new List<IntrareDjurnal>(); }
}
```

```csharp
// Salvare XML
XmlSerializer serializer = new XmlSerializer(typeof(JurnalWrapper));
JurnalWrapper wrapper = new JurnalWrapper { Intrari = intrari };
using (StreamWriter writer = new StreamWriter(cale))
    serializer.Serialize(writer, wrapper);

// Incarcare XML
using (StreamReader reader = new StreamReader(cale))
{
    JurnalWrapper wrapper = (JurnalWrapper)serializer.Deserialize(reader);
    intrari = wrapper.Intrari;
}
```

| Aspect            | `BinaryFormatter`             | `XmlSerializer`                 |
| ----------------- | ----------------------------- | ------------------------------- |
| **Format**        | Binar — neinteligibil         | XML — text lizibil              |
| **Stream**        | `FileStream`                  | `StreamWriter` / `StreamReader` |
| **Colecții**      | Serializează `List<T>` direct | Necesită clasă wrapper          |
| **Proprietăți**   | Toate câmpurile               | Doar publice                    |
| **Portabilitate** | Legat de .NET                 | Portabil                        |


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-9-serializarea-si-deserializarea/rezumat.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
