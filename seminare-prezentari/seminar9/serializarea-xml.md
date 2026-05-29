# Serializarea XML

`XmlSerializer` produce un fișier XML lizibil — text pur, structurat cu tag-uri, care poate fi deschis și inspectat în orice editor. Spre deosebire de `BinaryFormatter` care produce un flux de octeți opac, un fișier XML serializat arată astfel:

```xml
<?xml version="1.0" encoding="utf-8"?>
<JurnalWrapper>
  <Intrari>
    <IntrareDjurnal>
      <Data>2024-03-15T00:00:00</Data>
      <Titlu>Prima zi de primăvară</Titlu>
      <Continut>Azi a ieșit soarele...</Continut>
      <Dispozitie>Fericit</Dispozitie>
      <EstePrivata>false</EstePrivata>
    </IntrareDjurnal>
  </Intrari>
</JurnalWrapper>
```

Aceasta este principala diferență practică față de serializarea binară: fișierul XML poate fi citit, editat și chiar creat manual. Este mai verbose — ocupă mai mult spațiu — dar este portabil și inspectat ușor.

### `XmlSerializer` și clasa wrapper

`XmlSerializer` se găsește în `System.Xml.Serialization`. Spre deosebire de `BinaryFormatter`, care serializa `List<IntrareDjurnal>` direct, `XmlSerializer` are dificultăți cu colecțiile ca element rădăcină. Soluția standard este o clasă **wrapper** — o clasă simplă care conține lista:

```csharp
[Serializable]
public class JurnalWrapper
{
    public List<IntrareDjurnal> Intrari { get; set; }

    public JurnalWrapper()
    {
        Intrari = new List<IntrareDjurnal>();
    }
}
```

La salvare, înfășori lista în wrapper. La încărcare, extragi lista din wrapper. `XmlSerializer` serializează wrapper-ul, care conține lista.

### Cerințele pentru clasele serializate cu `XmlSerializer`

`XmlSerializer` este mai pretențios decât `BinaryFormatter` în privința structurii claselor:

**Constructor fără parametri** — obligatoriu pe orice clasă serializată, inclusiv `IntrareDjurnal` și `JurnalWrapper`. `XmlSerializer` creează obiectele apelând constructorul fără parametri și apoi populând proprietățile.

**Proprietăți publice** — `XmlSerializer` serializează doar proprietățile și câmpurile publice. Câmpurile private sunt ignorate.

**Atributul `[Serializable]`** — recomandat pe clase, dar `XmlSerializer` nu îl impune ca `BinaryFormatter`. Este bună practică să îl adaugi oricum.

Enum-urile sunt serializate automat ca string-uri cu numele valorii — `Fericit`, `Neutru` etc. Nu necesită configurare suplimentară.

### Salvarea cu `XmlSerializer`

Serializarea XML folosește `StreamWriter` în loc de `FileStream` — `StreamWriter` gestionează encoding-ul text corect:

```csharp
using System.IO;
using System.Xml.Serialization;

private void mniSalveazaXml_Click(object sender, EventArgs e)
{
    using (SaveFileDialog dialog = new SaveFileDialog())
    {
        dialog.Filter = "Jurnal XML (*.xml)|*.xml";
        dialog.DefaultExt = "xml";
        dialog.OverwritePrompt = true;

        if (dialog.ShowDialog() != DialogResult.OK) return;

        try
        {
            // Invelim lista in wrapper
            JurnalWrapper wrapper = new JurnalWrapper();
            wrapper.Intrari = intrari;

            XmlSerializer serializer = new XmlSerializer(typeof(JurnalWrapper));

            using (StreamWriter writer = new StreamWriter(dialog.FileName))
                serializer.Serialize(writer, wrapper);

            lblStatus.Text = "Salvat XML: " + Path.GetFileName(dialog.FileName);
        }
        catch (Exception ex)
        {
            MessageBox.Show("Eroare la salvarea XML:\n" + ex.Message,
                "Eroare", MessageBoxButtons.OK, MessageBoxIcon.Error);
        }
    }
}
```

`new XmlSerializer(typeof(JurnalWrapper))` creează un serializer specific tipului `JurnalWrapper`. `XmlSerializer` este generic prin constructor, nu prin tip generic — aceasta este o diferență față de alte API-uri .NET.

### Încărcarea cu `XmlSerializer`

Deserializarea XML folosește `StreamReader`:

```csharp
private void mniDeschideXml_Click(object sender, EventArgs e)
{
    using (OpenFileDialog dialog = new OpenFileDialog())
    {
        dialog.Filter = "Jurnal XML (*.xml)|*.xml";

        if (dialog.ShowDialog() != DialogResult.OK) return;

        try
        {
            XmlSerializer serializer = new XmlSerializer(typeof(JurnalWrapper));

            using (StreamReader reader = new StreamReader(dialog.FileName))
            {
                JurnalWrapper wrapper = (JurnalWrapper)serializer.Deserialize(reader);
                intrari = wrapper.Intrari;
            }

            RefreshLista();
            lblStatus.Text = "Deschis XML: " + Path.GetFileName(dialog.FileName) +
                             " (" + intrari.Count + " intrări)";
        }
        catch (Exception ex)
        {
            MessageBox.Show("Eroare la deschiderea XML:\n" + ex.Message,
                "Eroare", MessageBoxButtons.OK, MessageBoxIcon.Error);
        }
    }
}
```

### Comparație directă: `BinaryFormatter` vs. `XmlSerializer`

Cele două abordări diferă în câteva aspecte practice importante:

| Aspect                         | `BinaryFormatter`                 | `XmlSerializer`                                             |
| ------------------------------ | --------------------------------- | ----------------------------------------------------------- |
| **Formatul fișierului**        | Binar — neinteligibil pentru om   | XML — text lizibil în orice editor                          |
| **Dimensiunea fișierului**     | Mai compactă                      | Mai mare — tag-urile XML adaugă overhead                    |
| **Viteza**                     | Mai rapidă                        | Mai lentă pentru volume mari                                |
| **Stream folosit**             | `FileStream`                      | `StreamWriter` / `StreamReader`                             |
| **Clasa pentru colecții**      | Serializează `List<T>` direct     | Necesită clasă wrapper                                      |
| **Proprietăți serializate**    | Toate câmpurile, inclusiv private | Doar proprietăți și câmpuri publice                         |
| **Constructor fără parametri** | Necesar                           | Necesar                                                     |
| **Portabilitate**              | Legat de structura clasei .NET    | Portabil — poate fi citit de orice parser XML               |
| **Inspecție manuală**          | Imposibilă                        | Posibilă — fișierul poate fi deschis în browser sau Notepad |


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-9-serializarea-si-deserializarea/serializarea-xml.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
