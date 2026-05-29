# Serializarea binară

**Serializarea** este procesul de transformare a unui obiect din memorie într-o secvență de octeți care poate fi stocată pe disc. **Deserializarea** este procesul invers — reconstruirea obiectului din secvența de octeți.

Fără serializare, datele dintr-o aplicație trăiesc exclusiv în RAM și dispar la închidere. Cu serializare, poți salva starea completă a unui obiect — inclusiv toate câmpurile și colecțiile — și o poți restaura ulterior exact cum era.

### `[Serializable]` — marcarea clasei

`BinaryFormatter` poate serializa doar clasele marcate explicit cu atributul `[Serializable]`. Fără el, la serializare apare `SerializationException`. Atributul se pune deasupra declarației clasei:

```csharp
using System;

[Serializable]
public class IntrareDjurnal
{
    public DateTime Data { get; set; }
    public string Titlu { get; set; }
    public string Continut { get; set; }
    public Dispozitie Dispozitie { get; set; }
    public bool EstePrivata { get; set; }

    // Constructor fara parametri — necesar pentru deserializare
    public IntrareDjurnal() { }
}
```

Atributul trebuie aplicat pe **toate** clasele implicate în serializare. Enum-urile, `DateTime`, `string` și tipurile primitive din .NET sunt serializabile implicit — nu necesită `[Serializable]`.

Constructorul fără parametri este necesar deoarece `BinaryFormatter` creează obiectul la deserializare apelând constructorul fără parametri și apoi populând câmpurile. Dacă ai definit un alt constructor și lipsește cel fără parametri, deserializarea eșuează.

### Salvarea cu `BinaryFormatter`

`BinaryFormatter` se găsește în `System.Runtime.Serialization.Formatters.Binary`. Serializarea se face printr-un `FileStream` deschis pentru scriere:

```csharp
using System.IO;
using System.Runtime.Serialization.Formatters.Binary;

private void mniSalveaza_Click(object sender, EventArgs e)
{
    using (SaveFileDialog dialog = new SaveFileDialog())
    {
        dialog.Filter = "Jurnal binar (*.jrn)|*.jrn";
        dialog.DefaultExt = "jrn";
        dialog.OverwritePrompt = true;

        if (dialog.ShowDialog() != DialogResult.OK) return;

        try
        {
            BinaryFormatter formatter = new BinaryFormatter();
            using (FileStream stream = new FileStream(dialog.FileName, FileMode.Create))
                formatter.Serialize(stream, intrari);

            lblStatus.Text = "Salvat: " + Path.GetFileName(dialog.FileName);
        }
        catch (Exception ex)
        {
            MessageBox.Show("Eroare la salvare:\n" + ex.Message,
                "Eroare", MessageBoxButtons.OK, MessageBoxIcon.Error);
        }
    }
}
```

`FileMode.Create` creează fișierul dacă nu există sau îl suprascrie dacă există. `formatter.Serialize(stream, intrari)` scrie lista completă dintr-un singur apel — nu este nevoie să iterezi și să serializezi fiecare element separat.

### Încărcarea cu `BinaryFormatter`

Deserializarea citește fluxul de octeți și reconstruiește obiectul. `Deserialize` returnează `object`, care trebuie convertit la tipul corect:

```csharp
private void mniDeschide_Click(object sender, EventArgs e)
{
    using (OpenFileDialog dialog = new OpenFileDialog())
    {
        dialog.Filter = "Jurnal binar (*.jrn)|*.jrn";

        if (dialog.ShowDialog() != DialogResult.OK) return;

        try
        {
            BinaryFormatter formatter = new BinaryFormatter();
            using (FileStream stream = new FileStream(dialog.FileName, FileMode.Open))
                intrari = (List<IntrareDjurnal>)formatter.Deserialize(stream);

            RefreshLista();
            lblStatus.Text = "Deschis: " + Path.GetFileName(dialog.FileName) +
                             " (" + intrari.Count + " intrări)";
        }
        catch (Exception ex)
        {
            MessageBox.Show("Eroare la deschidere:\n" + ex.Message,
                "Eroare", MessageBoxButtons.OK, MessageBoxIcon.Error);
        }
    }
}
```

`FileMode.Open` deschide fișierul existent — dacă nu există, aruncă excepție. Deoarece `OpenFileDialog` cu `CheckFileExists = true` (implicit) nu permite selectarea unui fișier inexistent, această situație nu ar trebui să apară în practică, dar `try/catch` o gestionează oricum.

### Avantaje și limitări

Serializarea binară este rapidă, produce fișiere compacte și nu necesită nicio librărie externă — totul este în .NET Framework. Serializează automat orice structură de date marcată `[Serializable]`, inclusiv liste și obiecte imbricate.

Limitarea principală: fișierul binar este strâns legat de structura clasei. Dacă redenumești un câmp sau muți clasa în alt namespace, fișierele create cu versiunea anterioară nu mai pot fi deserializate. Pentru exercițiu, unde aplicația este scrisă și citită în aceeași sesiune, aceasta nu este o problemă.


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-9-serializarea-si-deserializarea/serializarea-binara.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
