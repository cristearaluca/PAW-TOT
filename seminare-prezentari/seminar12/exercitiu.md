# Exercițiu

Construiești o aplicație care vizualizează ierarhia angajaților unei companii. În partea stângă, un `TreeView` afișează angajații organizați ierarhic — fiecare nod reprezentând un angajat, cu subnoduri pentru subordonații direcți. La click pe un angajat, panoul din dreapta afișează detaliile complete ale acestuia și fotografia sa.

### Modelul de date și `FakeDatabase`

Declară enumerația `Departament` cu valorile: `Tehnic`, `Vanzari`, `HR`, `Financiar`, `Management`.

Declară clasa `Angajat` cu proprietățile:

* `Guid Id` — generat cu `Guid.NewGuid()` în constructor
* `string NumeComplet`
* `string Functie`
* `Departament Departament`
* `DateTime DataAngajarii`
* `decimal Salariu`
* `string Email`
* `Guid? ManagerId` — `null` pentru angajații fără manager (rădăcinile ierarhiei)
* `string NumeImagine` — numele fișierului imagine, ex. `"ion_popescu.jpg"`

Declară clasa statică `FakeDatabase` cu `static List<Angajat> Angajati` populată cu cel puțin 10 angajați pe 4–5 niveluri de ierarhie, cu `ManagerId`-urile corect setate.

### Fereastra principală

Pe `Form1` adaugă:

* Un `TreeView` (`tvAngajati`) în partea stângă, cu `Dock = Left` sau ancorat
* Un `Panel` (`pnlDetalii`) în partea dreaptă cu:
  * Un `PictureBox` (`picAngajat`) sus, cu `SizeMode = Zoom`
  * `Label`-uri pentru fiecare câmp: nume complet, funcție, departament, data angajării, salariu, email, și numele managerului direct
* Un `Splitter` între `TreeView` și panou pentru redimensionare

Imaginile se află într-un folder `Images` în directorul soluției. Calea se construiește în cod:

```csharp
string caleImagine = Path.Combine(
    Application.StartupPath, "Images", angajat.NumeImagine);
```

Dacă fișierul nu există, `PictureBox`-ul afișează o imagine placeholder `"default.jpg"` din același folder.

### Construirea ierarhiei în `TreeView`

Implementează metoda `IncarcaTreeView()` care construiește ierarhia recursiv din lista plată `FakeDatabase.Angajati`.

Algoritmul pornește de la angajații fără manager (`ManagerId == null`) — aceștia sunt rădăcinile arborelui. Pentru fiecare nod adăugat, se adaugă recursiv subordonații direcți.

Apelează `tvAngajati.ExpandAll()` după populare pentru a afișa toată ierarhia la pornire.

### Afișarea detaliilor la selecție

Abonează-te la evenimentul `AfterSelect` al `TreeView`-ului. La selecția unui nod, recuperează obiectul `Angajat` din `Tag` și populează panoul din dreapta:

### Date de test

```
using System;
using System.Collections.Generic;

namespace AngajatiApp
{   
    public static class FakeDatabase
    {
        private static readonly Guid idDirector        = Guid.NewGuid();
        private static readonly Guid idCtoCTO          = Guid.NewGuid();
        private static readonly Guid idVPVanzari       = Guid.NewGuid();
        private static readonly Guid idDirectorHR      = Guid.NewGuid();
        private static readonly Guid idLiderBackend    = Guid.NewGuid();
        private static readonly Guid idLiderFrontend   = Guid.NewGuid();
        private static readonly Guid idManagerVanzariN = Guid.NewGuid();
        private static readonly Guid idManagerVanzariS = Guid.NewGuid();
        private static readonly Guid idHRSenior        = Guid.NewGuid();

        public static List<Angajat> Angajati = new List<Angajat>
        {
            // ── Nivel 1 — Director General ───────────────────────────────────
            new Angajat
            {
                Id            = idDirector,
                NumeComplet   = "Alexandru Ionescu",
                Functie       = "Director General",
                Departament   = Departament.Management,
                DataAngajarii = new DateTime(2015, 3, 1),
                Salariu       = 25000m,
                Email         = "a.ionescu@company.ro",
                ManagerId     = null,
                NumeImagine   = "alexandru_ionescu.jpg"
            },

            // ── Nivel 2 — Directori de departament ──────────────────────────
            new Angajat
            {
                Id            = idCtoCTO,
                NumeComplet   = "Maria Popescu",
                Functie       = "Chief Technology Officer",
                Departament   = Departament.Tehnic,
                DataAngajarii = new DateTime(2016, 6, 15),
                Salariu       = 20000m,
                Email         = "m.popescu@company.ro",
                ManagerId     = idDirector,
                NumeImagine   = "maria_popescu.jpg"
            },
            new Angajat
            {
                Id            = idVPVanzari,
                NumeComplet   = "Bogdan Dumitrescu",
                Functie       = "VP Vânzări",
                Departament   = Departament.Vanzari,
                DataAngajarii = new DateTime(2017, 1, 10),
                Salariu       = 18000m,
                Email         = "b.dumitrescu@company.ro",
                ManagerId     = idDirector,
                NumeImagine   = "bogdan_dumitrescu.jpg"
            },
            new Angajat
            {
                Id            = idDirectorHR,
                NumeComplet   = "Elena Constantin",
                Functie       = "Director HR",
                Departament   = Departament.HR,
                DataAngajarii = new DateTime(2016, 9, 1),
                Salariu       = 16000m,
                Email         = "e.constantin@company.ro",
                ManagerId     = idDirector,
                NumeImagine   = "elena_constantin.jpg"
            },

            // ── Nivel 3 — Manageri de echipă ────────────────────────────────
            new Angajat
            {
                Id            = idLiderBackend,
                NumeComplet   = "Cristian Marin",
                Functie       = "Tech Lead Backend",
                Departament   = Departament.Tehnic,
                DataAngajarii = new DateTime(2018, 4, 1),
                Salariu       = 14000m,
                Email         = "c.marin@company.ro",
                ManagerId     = idCtoCTO,
                NumeImagine   = "cristian_marin.jpg"
            },
            new Angajat
            {
                Id            = idLiderFrontend,
                NumeComplet   = "Andreea Stanescu",
                Functie       = "Tech Lead Frontend",
                Departament   = Departament.Tehnic,
                DataAngajarii = new DateTime(2019, 2, 15),
                Salariu       = 13500m,
                Email         = "a.stanescu@company.ro",
                ManagerId     = idCtoCTO,
                NumeImagine   = "andreea_stanescu.jpg"
            },
            new Angajat
            {
                Id            = idManagerVanzariN,
                NumeComplet   = "Radu Popa",
                Functie       = "Manager Vânzări Nord",
                Departament   = Departament.Vanzari,
                DataAngajarii = new DateTime(2018, 7, 1),
                Salariu       = 12000m,
                Email         = "r.popa@company.ro",
                ManagerId     = idVPVanzari,
                NumeImagine   = "radu_popa.jpg"
            },
            new Angajat
            {
                Id            = idManagerVanzariS,
                NumeComplet   = "Ioana Dragomir",
                Functie       = "Manager Vânzări Sud",
                Departament   = Departament.Vanzari,
                DataAngajarii = new DateTime(2019, 3, 1),
                Salariu       = 12000m,
                Email         = "i.dragomir@company.ro",
                ManagerId     = idVPVanzari,
                NumeImagine   = "ioana_dragomir.jpg"
            },
            new Angajat
            {
                Id            = idHRSenior,
                NumeComplet   = "Mihai Florescu",
                Functie       = "HR Senior Specialist",
                Departament   = Departament.HR,
                DataAngajarii = new DateTime(2019, 5, 1),
                Salariu       = 10000m,
                Email         = "m.florescu@company.ro",
                ManagerId     = idDirectorHR,
                NumeImagine   = "mihai_florescu.jpg"
            },

            // ── Nivel 4 — Angajați ──────────────────────────────────────────
            new Angajat
            {
                Id            = Guid.NewGuid(),
                NumeComplet   = "George Nistor",
                Functie       = "Senior Backend Developer",
                Departament   = Departament.Tehnic,
                DataAngajarii = new DateTime(2020, 1, 15),
                Salariu       = 9000m,
                Email         = "g.nistor@company.ro",
                ManagerId     = idLiderBackend,
                NumeImagine   = "george_nistor.jpg"
            },
            new Angajat
            {
                Id            = Guid.NewGuid(),
                NumeComplet   = "Simona Badea",
                Functie       = "Backend Developer",
                Departament   = Departament.Tehnic,
                DataAngajarii = new DateTime(2021, 6, 1),
                Salariu       = 7500m,
                Email         = "s.badea@company.ro",
                ManagerId     = idLiderBackend,
                NumeImagine   = "simona_badea.jpg"
            },
            new Angajat
            {
                Id            = Guid.NewGuid(),
                NumeComplet   = "Vlad Ionescu",
                Functie       = "Frontend Developer",
                Departament   = Departament.Tehnic,
                DataAngajarii = new DateTime(2021, 9, 1),
                Salariu       = 7000m,
                Email         = "v.ionescu@company.ro",
                ManagerId     = idLiderFrontend,
                NumeImagine   = "vlad_ionescu.jpg"
            },
            new Angajat
            {
                Id            = Guid.NewGuid(),
                NumeComplet   = "Larisa Toma",
                Functie       = "Agent Vânzări",
                Departament   = Departament.Vanzari,
                DataAngajarii = new DateTime(2020, 3, 1),
                Salariu       = 6000m,
                Email         = "l.toma@company.ro",
                ManagerId     = idManagerVanzariN,
                NumeImagine   = "larisa_toma.jpg"
            },
            new Angajat
            {
                Id            = Guid.NewGuid(),
                NumeComplet   = "Daniel Gheorghe",
                Functie       = "Agent Vânzări",
                Departament   = Departament.Vanzari,
                DataAngajarii = new DateTime(2022, 1, 10),
                Salariu       = 5800m,
                Email         = "d.gheorghe@company.ro",
                ManagerId     = idManagerVanzariS,
                NumeImagine   = "daniel_gheorghe.jpg"
            },
            new Angajat
            {
                Id            = Guid.NewGuid(),
                NumeComplet   = "Ana Rusu",
                Functie       = "HR Specialist",
                Departament   = Departament.HR,
                DataAngajarii = new DateTime(2022, 4, 1),
                Salariu       = 6500m,
                Email         = "a.rusu@company.ro",
                ManagerId     = idHRSenior,
                NumeImagine   = "ana_rusu.jpg"
            },

            // ── Nivel 5 — Junior ────────────────────────────────────────────
            new Angajat
            {
                Id            = Guid.NewGuid(),
                NumeComplet   = "Tudor Petrescu",
                Functie       = "Junior Backend Developer",
                Departament   = Departament.Tehnic,
                DataAngajarii = new DateTime(2023, 9, 1),
                Salariu       = 5500m,
                Email         = "t.petrescu@company.ro",
                ManagerId     = idLiderBackend,
                NumeImagine   = "tudor_petrescu.jpg"
            },
        };
    }
}
```
