# Cuvintele cheie virtual și override

Moștenirea, prezentată în capitolul anterior, transmite automat metodele clasei de bază către clasele derivate. Aceasta este exact ce dorim când comportamentul este identic (de exemplu, verificarea că numărul de loc este între 1 și 200 este aceeași pentru orice tip de bilet).

Dar există metode al căror comportament trebuie să **difere** de la un tip la altul. Clasa `Bilet` are o metodă `GetReducere()` care returnează `0`, deoarece un bilet standard nu beneficiază de reducere. `BiletStudent` trebuie să returneze 20% din prețul de bază. Dacă nu intervenim, `BiletStudent` moștenește implementarea din `Bilet` și returnează tot `0`, ignorând logica specifică studentului.

Avem nevoie de un mecanism prin care clasa derivată să **înlocuiască** o metodă moștenită cu propria ei versiune. Acesta este rolul perechii `virtual` / `override`.

### `virtual` - metoda care poate fi înlocuită

Modificatorul `virtual` marchează o metodă ca putând fi înlocuită de clasele derivate. Metoda are o implementare implicită și funcționează ca atare pentru orice clasă care nu alege să o suprascrie, dar această implementare **nu este definitivă**:

```csharp
class Bilet
{
    public double PretBaza { get; set; }

    public virtual double GetReducere()
    {
        return 0;  // implementare implicita: nicio reducere
    }

    public virtual bool EsteValid()
    {
        return ExpiraLa > DateTime.Now && NumarLoc >= 1 && NumarLoc <= 200;
    }
}
```

O metodă fără `virtual` nu poate fi înlocuită corect. Dacă o clasă derivată încearcă să o redefinească, compilatorul avertizează că **ascunde** metoda de bază fără să o înlocuiască, iar comportamentul în polimorfism va fi greșit.

### `override` - înlocuirea metodei din clasa derivată

`override` declară că metoda înlocuiește intenționat o metodă `virtual` din clasa de bază. Semnătura trebuie să fie identică: același nume, aceiași parametri, același tip de retur.

```csharp
class BiletStudent : Bilet
{
    public string NumarLegitimatie { get; set; }

    public override double GetReducere()
    {
        return PretBaza * 0.20;  // 20% din pretul de baza
    }

    public override bool EsteValid()
    {
        // extindem validarea din clasa de baza cu o conditie proprie
        return base.EsteValid() && !string.IsNullOrEmpty(NumarLegitimatie);
    }
}
```

Observați utilizarea `base.EsteValid()` în al doilea exemplu. `base` permite **apelarea implementării din clasa de bază** înainte de a adăuga logica suplimentară. Astfel nu duplicăm verificările deja existente în `Bilet`, ci le extindem cu condiția specifică studentului. Dacă validarea de bază se modifică ulterior, modificarea se propagă automat și în `BiletStudent`.

### Ce se întâmplă fără `virtual`?

Fără `virtual`, compilatorul leagă apelul metodei de **tipul variabilei**, nu de tipul real al obiectului din memorie. Aceasta se numește **legare statică** și produce rezultate incorecte atunci când lucrezi cu variabile de tip clasă de bază care stochează obiecte derivate:

```csharp
class Bilet
{
    // fara virtual — legare statica
    public double GetReducere() { return 0; }
}

class BiletStudent : Bilet
{
    // compilatorul avertizeaza: aceasta metoda ascunde GetReducere din Bilet
    public double GetReducere() { return PretBaza * 0.20; }
}
```

```csharp
BiletStudent bs = new BiletStudent(...);
Console.WriteLine(bs.GetReducere());  // 6 RON — corect, variabila este BiletStudent

Bilet b = new BiletStudent(...); 
Console.WriteLine(b.GetReducere());   // 0 RON — gresit, ignora implementarea din BiletStudent
```

Al doilea apel returnează `0` deoarece compilatorul privește tipul variabilei `b` (care este `Bilet`) și leagă apelul de implementarea din `Bilet`, ignorând complet că obiectul real din memorie este un `BiletStudent`.

Cu `virtual` și `override`, al doilea apel ar returna tot `6 RON`. C# se uită la **tipul real** al obiectului din memorie, nu la tipul declarat al variabilei.

### Legarea dinamică și baza polimorfismului

Mecanismul prin care C# alege, la momentul execuției, ce implementare să apeleze se numește **legare dinamică** sau **dispatch virtual**. Este baza polimorfismului, subiectul tratat în profunzime în capitolul următor.

Când scriem:

```csharp
List<Bilet> bilete = new List<Bilet>
{
    new Bilet(...),
    new BiletStudent(...),
    new BiletSenior(...),
    new BiletVIP(...)
};

foreach (Bilet b in bilete)
{
    Console.WriteLine(b.GetReducere());
}
```

La fiecare iterație, variabila `b` este de tip `Bilet`. Dar obiectul din memorie la care `b` pointează este de tipul real cu care a fost creat. C# caută implementarea `GetReducere()` pe tipul real al obiectului, nu pe tipul variabilei. Rezultatele vor fi diferite pentru fiecare element, conform propriei implementări. Vom detalia acest comportament în capitolul despre polimorfism.

### Apelul `base` - reutilizarea logicii din clasa de bază

Cuvântul cheie `base` permite accesul la membrii clasei de bază din interiorul clasei derivate. Cel mai frecvent este utilizat în metodele `override` pentru a apela implementarea originală înainte de a adăuga logică proprie:

```csharp
public override bool EsteValid()
{
    // Apelam mai intai validarea din Bilet
    if (!base.EsteValid())
        return false;

    // Adaugam conditia specifica BiletStudent
    return !string.IsNullOrEmpty(NumarLegitimatie);
}
```

Această abordare respectă principiul că clasa derivată **extinde** clasa de bază, nu o înlocuiește complet. Logica din `Bilet` rămâne activă. `BiletStudent` adaugă doar ce îi este specific.

### Lanțuri de suprascrieri

O metodă marcată cu `override` poate fi la rândul ei suprascrisă în clasele derivate ale derivatei. Comportamentul se propagă corect pe toată ierarhia:

```csharp
class BiletVIP : Bilet
{
    public bool IncludePopcorn { get; set; }
    public bool IncludeBautura { get; set; }

    public double GetExtras()
    {
        double extras = 0;
        if (IncludePopcorn) extras += 15;
        if (IncludeBautura) extras += 10;
        return extras;
    }

    public override double GetReducere() { return 0; }

    public override double CalculeazaPretFinal()
    {
        return PretBaza + GetExtras();  // VIP plateste baza + extras, fara reducere
    }
}

class BiletStudentVIP : BiletVIP
{
    // suprascrie din nou, combina logica VIP cu reducerea de student
    public override double CalculeazaPretFinal()
    {
        return PretBaza - (PretBaza * 0.10) + GetExtras();
    }
}
```

Ierarhia apelurilor urmează ierarhia claselor. `BiletStudentVIP` suprascrie `CalculeazaPretFinal` definită în `BiletVIP`, care la rândul ei suprascria cea din `Bilet`. La fiecare nivel, `override` înseamnă „aceasta este versiunea mea a acestei metode".

### `sealed override`

Dacă dorim să permitem unui nivel să suprascrie o metodă, dar să interzicem nivelurilor ulterioare să facă același lucru, combinăm `sealed` cu `override`:

```csharp
class BiletSenior : Bilet
{
    public sealed override double GetReducere()
    {
        return PretBaza * 0.30;
    }
}

class BiletSeniorPremium : BiletSenior
{
    public override double GetReducere() { ... }  // eroare de compilare
}
```

`sealed override` se utilizează rar, dar există situații în care logica unei metode la un anumit nivel al ierarhiei nu trebuie modificată mai departe.

### `abstract` ca alternativă

`virtual` oferă o implementare implicită pe care clasele derivate o pot înlocui. Există situații în care nu există o implementare implicită sensibilă și dorim să **forțăm** fiecare clasă derivată să definească ea propria implementare. Aceasta este situația metodelor `abstract`:

```csharp
abstract class Bilet
{
    // nu există implementare implicita — fiecare derivata TREBUIE sa o defineasca
    public abstract double GetReducere();
}
```

O clasă care conține cel puțin o metodă `abstract` trebuie declarată ea însăși ca `abstract` și nu poate fi instanțiată direct cu `new`. În exercițiul nostru nu folosim `abstract` deoarece `Bilet` are un comportament implicit valid (`GetReducere()` returnează `0`), dar conceptul este frecvent în cod real și merită cunoscut.


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-2-programarea-orientata-obiect-in-c/cuvintele-cheie-virtual-si-override.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
