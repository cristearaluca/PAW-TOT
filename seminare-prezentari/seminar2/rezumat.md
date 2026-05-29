# Rezumat

Această pagină sintetizează conceptele acoperite în toate capitolele. Nu înlocuiește lectura capitolelor, ci servește ca referință rapidă în timpul exercițiului practic.

### 1. Clasa și obiectul

| Concept         | Ce este                                         |
| --------------- | ----------------------------------------------- |
| **Clasă**       | Șablonul — descrie structura și comportamentul  |
| **Obiect**      | Instanța — un exemplar concret creat din șablon |
| **Proprietate** | O dată aparținând obiectului                    |
| **Metodă**      | Un comportament aparținând obiectului           |
| **`new`**       | Operatorul care creează un obiect în memorie    |

O variabilă de tip clasă conține o referință către obiect, nu obiectul în sine. Atribuirea `c2 = c1` nu copiază obiectul, ambele variabile pointează către același obiect din memorie.

### 2. Proprietăți și încapsulare

| Concept                           | Ce înseamnă                                                        |
| --------------------------------- | ------------------------------------------------------------------ |
| **Câmp privat**                   | Memoria internă a obiectului, inaccesibilă din exterior            |
| **Proprietate**                   | Poarta de acces controlată la acea memorie                         |
| **`get`**                         | Rulează când citim valoarea                                        |
| **`set`**                         | Rulează când scriem valoarea; `value` este valoarea nouă           |
| **Proprietate auto-implementată** | `{ get; set; }` — sintaxă scurtă când nu e nevoie de validare      |
| **Encapsulare**                   | Principiul că obiectul își controlează și protejează propria stare |

Câmpurile sunt `private`, proprietățile sunt `public`. Validarea pusă în `set` rulează automat la orice atribuire, inclusiv din constructor.

### 3. Constructorul

| Concept                        | Ce înseamnă                                                                    |
| ------------------------------ | ------------------------------------------------------------------------------ |
| **Constructor**                | Metodă specială apelată automat la `new`, fără tip de retur                    |
| **Parametrii constructorului** | Forțează furnizarea datelor la creare                                          |
| **Validare în proprietăți**    | Rulează automat prin constructor, fără apel explicit                           |
| **Supraîncărcare**             | Mai mulți constructori cu parametri diferiți                                   |
| **`: this(...)`**              | Apelează alt constructor din aceeași clasă                                     |
| **`: base(...)`**              | Apelează constructorul clasei de bază                                          |
| **Constructor implicit**       | Generat automat dacă nu există niciunul definit; dispare când definești primul |

Dacă constructorul aruncă excepție, obiectul nu ajunge în memorie. Un obiect invalid nu ar trebui să existe.

### 4. Enumerații

| Concept                     | Ce înseamnă                                          |
| --------------------------- | ---------------------------------------------------- |
| **`enum`**                  | Tip cu mulțime fixă de valori cunoscute la compilare |
| **Valoare implicită**       | Primul element are valoarea `0`, restul în ordine    |
| **Verificare la compilare** | Orice valoare în afara mulțimii produce eroare       |
| **`switch` pe enum**        | Mod natural de a trata fiecare caz în parte          |
| **`.ToString()`**           | Returnează numele valorii ca string                  |
| **`Enum.TryParse`**         | Conversie sigură din string în enum                  |

Enumerațiile înlocuiesc string-urile sau numerele magice cu un tip pe care compilatorul îl poate verifica.

### 5. Excepții

| Concept                         | Ce înseamnă                                                       |
| ------------------------------- | ----------------------------------------------------------------- |
| **`throw`**                     | Aruncă o excepție și oprește execuția curentă                     |
| **`ArgumentException`**         | Parametru primit din exterior cu valoare inacceptabilă            |
| **`InvalidOperationException`** | Operație imposibilă în starea curentă a obiectului                |
| **Propagare pe stivă**          | Excepția urcă prin apeluri până e prinsă sau programul se oprește |
| **`try/catch`**                 | Prinde excepția și permite continuarea programului                |
| **`finally`**                   | Rulează indiferent dacă a apărut excepție sau nu                  |
| **Constructor + excepție**      | Dacă constructorul aruncă excepție, obiectul nu există în memorie |

Excepțiile nu sunt mesaje de eroare pentru utilizator. Sunt mecanismul prin care codul semnalează că ceva fundamental nu poate continua.

### 6. Moștenirea

| Concept                     | Ce înseamnă                                                         |
| --------------------------- | ------------------------------------------------------------------- |
| **Moștenire**               | Clasa derivată primește tot ce e public/protected din clasa de bază |
| **`: ClasaDeBaza`**         | Sintaxa de declarare a moștenirii                                   |
| **`: base(...)`**           | Apelul constructorului clasei de bază                               |
| **Relația "este un"**       | Testul conceptual pentru a valida o moștenire                       |
| **Moștenire pe niveluri**   | O clasă derivată poate fi la rândul ei extinsă                      |
| **O singură clasă de bază** | C# nu permite moștenire multiplă din clase                          |
| **`sealed`**                | Interzice moștenirea unei clase                                     |

Moștenirea exprimă relații reale din domeniu. Testul: dacă propoziția "X este un Y" are sens, X poate moșteni Y.

### 7. `virtual` și `override`

| Concept               | Ce înseamnă                                                                    |
| --------------------- | ------------------------------------------------------------------------------ |
| **`virtual`**         | Metoda poate fi înlocuită de clasele derivate                                  |
| **`override`**        | Înlocuiește intenționat o metodă `virtual` din clasa de bază                   |
| **`base.Metoda()`**   | Apelează implementarea din clasa de bază înainte de a adăuga logică nouă       |
| **Legare dinamică**   | C# alege implementarea la runtime după tipul real al obiectului                |
| **Fără `virtual`**    | Legare statică — tipul variabilei decide, polimorfismul nu funcționează        |
| **`sealed override`** | Permite suprascrierea la nivelul curent, o interzice mai departe               |
| **`abstract`**        | Nu există implementare implicită; clasa derivată este obligată să o definească |

Fără `virtual`, apelul metodei pe o variabilă de tip bază ajunge mereu la implementarea clasei de bază, indiferent de tipul real al obiectului.

### 8. Polimorfismul

| Concept                             | Ce înseamnă                                                                    |
| ----------------------------------- | ------------------------------------------------------------------------------ |
| **Polimorfism**                     | Același apel produce rezultate diferite în funcție de tipul real al obiectului |
| **Tipul real vs. tipul variabilei** | C# caută implementarea pe tipul din memorie, nu pe tipul declarat              |
| **`List<Bilet>` cu obiecte mixte**  | Toate tipurile derivate sunt acceptate; fiecare răspunde cu propria logică     |
| **Extensibilitate**                 | Tipuri noi adăugate ulterior funcționează fără a modifica codul existent       |
| **Polimorfism prin interfețe**      | Funcționează la fel; obiectele sunt văzute prin tipul interfeței               |
| **`is`/`as`**                       | Pentru cazurile în care tipul concret chiar contează                           |

Polimorfismul este motivul pentru care `CasaBilete` poate fi scrisă simplu și rămâne corectă indiferent câte tipuri de bilete adăugăm.

### 9. Interfețele

| Concept                        | Ce înseamnă                                                                 |
| ------------------------------ | --------------------------------------------------------------------------- |
| **Interfață**                  | Contract care impune ce metode trebuie să existe, fără implementare         |
| **`interface`**                | Cuvântul cheie pentru declarare                                             |
| **Implementare**               | Clasa declară cu `:` și definește toate metodele din interfață              |
| **Moștenire vs. interfață**    | Moștenire: "este un", cod reutilizat; Interfață: "poate face", contract pur |
| **Oricâte interfețe**          | O clasă poate implementa mai multe interfețe simultan                       |
| **Decuplare**                  | Codul care consumă obiectele depinde de interfață, nu de tipul concret      |
| **Polimorfism prin interfață** | Funcționează identic cu polimorfismul prin moștenire                        |

Interfețele permit adăugarea de tipuri noi fără modificarea codului existent, atâta timp cât noile tipuri respectă contractele stabilite.

### 10. Genericele și `where`

| Concept                    | Ce înseamnă                                                                     |
| -------------------------- | ------------------------------------------------------------------------------- |
| **Parametru de tip `<T>`** | Placeholder înlocuit cu un tip concret la apel                                  |
| **`where T : Bilet`**      | Constrângere: `T` trebuie să fie `Bilet` sau derivat din el                     |
| **Fără `where`**           | Compilatorul nu știe nimic despre `T`; nu poate verifica proprietăți sau metode |
| **`OfType<T>()`**          | Filtrează o colecție și returnează elementele de tipul `T`                      |
| **Clasă generică**         | O clasă parametrizată cu un tip; exemplul clasic este `List<T>`                 |
| **Beneficiul principal**   | O singură implementare pentru orice tip, fără duplicare de cod                  |

Genericele permit scrierea de cod care funcționează cu orice tip fără să piardă verificările compilatorului.

### 11. LINQ

| Metodă                          | Ce face                                                |
| ------------------------------- | ------------------------------------------------------ |
| `Where`                         | Filtrează după o condiție                              |
| `OfType<T>`                     | Filtrează după tipul real al obiectului                |
| `Select`                        | Transformă fiecare element                             |
| `Sum`                           | Suma unei valori numerice                              |
| `Count`                         | Numără elementele, opțional cu condiție                |
| `Min` / `Max`                   | Valoarea minimă sau maximă                             |
| `Average`                       | Media aritmetică                                       |
| `OrderBy` / `OrderByDescending` | Sortare crescătoare sau descrescătoare                 |
| `ThenBy` / `ThenByDescending`   | Criteriu secundar de sortare                           |
| `First` / `FirstOrDefault`      | Primul element, cu sau fără excepție la colecție goală |
| `Single` / `SingleOrDefault`    | Exact un element, cu sau fără excepție                 |
| `Take`                          | Primele `n` elemente                                   |
| `ToList` / `ToArray`            | Materializează rezultatul într-o colecție concretă     |

Metodele LINQ pot fi înlănțuite. Rezultatul fiecăreia devine intrarea următoarei. `ToList()` la final materializează rezultatul într-o colecție concretă.

### 12. `is` și `as`

| Concept                        | Ce face                 | La eșec                                    |
| ------------------------------ | ----------------------- | ------------------------------------------ |
| **`(Tip)obiect`**              | Cast explicit           | Aruncă `InvalidCastException`              |
| **`obiect as Tip`**            | Conversie sigură        | Returnează `null`                          |
| **`obiect is Tip`**            | Verificare tip          | Returnează `false`                         |
| **`obiect is Tip var`**        | Verificare și extragere | Returnează `false`; variabila inaccesibilă |
| **`switch` cu `case Tip var`** | Ramificare pe tip       | Merge pe `default`                         |

Pattern matching cu `is` este varianta modernă și preferată. Ordinea ramurilor contează când tipurile sunt din aceeași ierarhie: tipurile mai specifice se verifică primele.

### Cum se leagă toate

```
Interfete: IPretCalculabil, IValidabil
  → definesc contractul, nu implementarea
  → implementate de clasa de baza

Clasa de baza: Bilet
  → encapsulare: campuri private, proprietati cu validare
  → constructor: obiect valid de la creare sau deloc
  → metode virtual: GetReducere(), EsteValid(), CalculeazaPretFinal()

Clase derivate: BiletStudent, BiletSenior, BiletVIP
  → mostenire: preiau tot din Bilet
  → override: isi definesc propria logica de reducere si validare
  → constructori cu :base(...): initializeaza Bilet, adauga proprii parametri

Polimorfism
  → List<Bilet> cu obiecte mixte
  → acelasi apel, rezultate diferite dupa tipul real
  → extensibil: tipuri noi fara modificarea codului existent

CasaBilete
  → List<Bilet>: polimorfism in actiune
  → LINQ: Sum, OrderByDescending, Count
  → GetNumarBiletePerTip<T>(): generice + OfType<T>
  → is/as: cand tipul concret conteaza
```


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-2-programarea-orientata-obiect-in-c/rezumat.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
