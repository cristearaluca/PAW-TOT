# Exercițiu

Scrieți un program consolă care definește o clasă `Student` cu următoarele proprietăți:&#x20;

* `Nume (string)`
* `Grupa (string)`
* `NotaExamen (double?)`
* `NotaLaborator (double?)`

Clasa trebuie să aibă o proprietate calculată `Media` care returnează media celor două note (sau `null` dacă una dintre ele lipsește).&#x20;

Clasa trebuie să aibă o metodă `Afiseaza()` care afișează informațiile studentului folosind string interpolation.

În metoda `Main`, creați un array de cel puµin 4 studenți (unii cu note, alții fără). Folosiți `foreach` pentru a-i afișa pe toți.&#x20;

Dacă toți studenții au note, calculați și afișați numărul de studenți promovați (media ≥ 5) și media generală a grupei, folosind operatorul ?? pentru a trata notele lipsă.


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-1-introducere-in-c-si-.net/exercitiu.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
