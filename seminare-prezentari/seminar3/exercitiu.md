# Exercițiu

Implementați un sistem simplificat de urmărire a comenzilor pentru un magazin online. O comandă trece prin mai multe stări: `Plasata`, `Procesata`, `Expediata`, `Livrata`. La fiecare tranziție, sistemul notifică automat clientul prin mai multe canale. Costul de livrare se calculează după strategii diferite, configurabile la runtime.

### Structura de bază

Definiți enumerația `StareComanda` cu valorile: `Plasata`, `Procesata`, `Expediata`, `Livrata`.

Definiți clasa `Produs` cu proprietățile:

* `string Nume`
* `decimal Pret`
* `int Cantitate`

Definiți clasa `Comanda` cu proprietățile readonly, setate la constructor:

* `string NumarComanda`, generat automat în constructor, de forma `CMD-001`, `CMD-002` etc.
* `string NumeClient`
* `string EmailClient`
* `List<Produs> Produse`, inițializată în constructor cu o listă goală
* `StareComanda Stare`, inițializată cu `Plasata` - poate fi setată și din exteriorul clasei
* `DateTime DataPlasare`, setată automat în constructor cu `DateTime.Now`

Adăugați metodele:

* `decimal GetValoareTotala()`, suma `Pret * Cantitate` pentru toate produsele
* `void AdaugaProdus(Produs p)`, adaugă un produs în listă
* `override string ToString()`, returnează un șir cu numărul comenzii, clientul și valoarea totală

### Delegat și notificări multicast

Declarați un delegat numit `NotificareClient` care primește doi parametri (`string numarComanda`, `string mesaj`) și nu returnează nimic.

Implementați în clasa statică `NotificareService` trei metode statice compatibile cu acest delegat:

* `NotificaPrinEmail(string numarComanda, string mesaj)` — afișează prefixat cu `[EMAIL]`
* `NotificaPrinSMS(string numarComanda, string mesaj)` — afișează prefixat cu `[SMS]`
* `Log(string numarComanda, string mesaj)` — afișează prefixat cu `[LOG]`

Demonstrați în `Main`:

* Crearea unui delegat multicast care include toate trei metodele
* Invocarea lui cu un mesaj de test
* Eliminarea `NotificareSMS` cu `-=` și demonstrarea că nu mai este apelată

### `Action`&#x20;

Refactorizați sistemul de notificări pentru a folosi `Action<string, string>` în loc de delegatul `NotificareClient`. Adăugați și o notificare push scrisă direct ca lambda la momentul abonării.

### Clasa `Depozit` cu evenimente

Definiți clasa `Depozit` care procesează comenzi și gestionează tranzițiile de stare.

**Clasele `EventArgs`:**

`ComandaSchimbatStareEventArgs` — derivată din `EventArgs`, conține:

* `Comanda Comanda`
* `StareComanda StareVeche`
* `StareComanda StareNoua`

`ComandaLivrataEventArgs` — derivată din `EventArgs`, conține:

* `Comanda Comanda`
* `DateTime DataLivrare`

**Proprietăți și evenimente în `Depozit`:**

* `string NumeDepozit`
* `event EventHandler<ComandaSchimbatStareEventArgs> ComandaSchimbatStare`
* `event EventHandler<ComandaLivrataEventArgs> ComandaLivrata`

**Metode:**

`InregistreazaComanda(Comanda c, NotificareClient onNotificareClient = null)` adaugă comanda în colecția internă privată și notifică clientul prin intermediul unui delegat

`AvansezaStare(string numarComanda)` — găsește comanda după număr, avansează starea la următoarea valoare din enumerație și declanșează `ComandaSchimbatStare`; dacă noua stare este `Livrata`, declanșează și `ComandaLivrata`

`GetComenziActive()` — returnează comenzile care nu au starea `Livrata`

### Abonare la evenimente

În `Main`, creați un `Depozit`, înregistrați cel puțin 3 comenzi și abonați-vă la ambele evenimente.

Avansați fiecare comandă prin toate stările apelând `AvansezaStare` în buclă și observați că:

* `ComandaSchimbatStare` se declanșează la fiecare tranziție
* `ComandaLivrata` se declanșează doar la starea finală


---

# Agent Instructions: Querying This Documentation

If you need additional information that is not directly available in this page, you can query the documentation dynamically by asking a question.

Perform an HTTP GET request on the current page URL with the `ask` query parameter:

```
GET https://ase.lucianvilcea.ro/programarea-aplicatiilor-windows/seminar-3-delegati-actiuni-si-evenimente/exercitiu.md?ask=<question>
```

The question should be specific, self-contained, and written in natural language.
The response will contain a direct answer to the question and relevant excerpts and sources from the documentation.

Use this mechanism when the answer is not explicitly present in the current page, you need clarification or additional context, or you want to retrieve related documentation sections.
