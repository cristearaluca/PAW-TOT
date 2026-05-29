# Exercițiu

Implementați un sistem simplificat de vânzare a biletelor pentru un cinema. Sistemul gestionează mai multe tipuri de bilete, fiecare cu propriul mod de calcul al prețului, și o casă de bilete care centralizează vânzările.

### Clasa `Client`

Definiți clasa `Client` cu următoarele proprietăți:

* `string Nume`&#x20;
* `string Prenume`&#x20;
* `string Email` – conține caracterele `@` și `.`
* `string Telefon` – exact 10 caractere

### Interfețe

Definiți două interfețe:

`IPretCalculabil` cu metodele:

* `double CalculeazaPretFinal()` — returnează prețul pe care clientul îl plătește efectiv
* `double GetReducere()` — returnează suma scăzută din prețul de bază (nu procentul, ci valoarea în RON)

`IValidabil` cu metoda:

* `bool EsteValid()` — returnează true dacă biletul este în perioada de valabilitate

### Clasa `Bilet`

Definiți clasa `Bilet` care implementează ambele interfețe de mai sus.

Clasa conține următoarele date:

* `string NumeFilm`
* `int NumarSala`
* `TipFilm TipFilm` – enumerație care conține valorile `Comedie`, `Acțiune`, `Dramă`, `Istoric`
* `DateTime ExpiraLa`
* `Client Client`
* `int NumarLoc` – numărul locului, între 1 și 200; dacă valoarea transmisă este în afara acestui interval, se aruncă `ArgumentException`
* `double PretBaza` – prețul fără reduceri, strict pozitiv; dacă valoarea transmisă este negativă sau zero, se aruncă `ArgumentException`

Constructorul primește toate câmpurile de mai sus ca parametri.

Comportament așteptat:

* `GetReducere()` returnează `0` - biletul standard nu are reducere
* `CalculeazaPretFinal()` returnează `PretBaza - GetReducere()`
* `EsteValid()` returnează true doar dacă ExpiraLa este în viitor și NumarLoc este valid

### Clase derivate

`BiletStudent` - derivată din `Bilet`

Date suplimentare:

* `string NumarLegitimatie`
* `string Facultate`

Comportament așteptat:

* Reducerea este de 20% din prețul de bază
* `EsteValid()` returnează `false` și dacă numărul legitimației este gol sau `null`

`BiletSenior` - derivată din `Bilet`

Date suplimentare:&#x20;

* `int VarstaClient`

Comportament așteptat:

* Dacă vârsta transmisă în constructor este sub 60 de ani, se aruncă `ArgumentException`
* Reducerea este de 30% din prețul de bază

`BiletVIP` - derivată din `Bilet`

Date suplimentare:&#x20;

* `bool IncludePopcorn`
* `bool IncludeBautura`&#x20;

Comportament așteptat:

* Biletul VIP nu are reducere
* Popcorn-ul costă 15 RON, băutura costă 10 RON
* Definiți metoda `GetExtras()` care returnează suma serviciilor suplimentare alese
* `CalculeazaPretFinal()` returnează `PretBaza + GetExtras()`

### Clasa `CasaBilete`

Definiți clasa `CasaBilete` care gestionează o colecție privată de bilete vândute.

Metode necesare:

* `AdaugaBilet(Bilet b)` — adaugă biletul în colecție
* `GetIncasariTotale()` — returnează suma prețurilor finale ale tuturor biletelor din colecție
* `GetReduceriAcordate()` — returnează suma totală a reducerilor acordate
* `GetNumarBiletePerTip<T>()` — returnează numărul de bilete de un anumit tip
* `GetBiletulCelMaiScump()` — returnează biletul cu cel mai mare preț final


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-2-programarea-orientata-obiect-in-c/exercitiu.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
