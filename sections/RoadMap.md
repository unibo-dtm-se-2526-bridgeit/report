# Roadmap del Progetto e Assegnazione Task

Questa roadmap definisce gli obiettivi settimanali per il team e **opera concretamente le Milestone già definite in `report.md` (sezione Roadmap)**: non è un piano alternativo, ma la traduzione di quelle milestone in task settimanali assegnati a persona, con sub-task.

**Squadra:**
- `@nikytresca` — Sviluppatrice di moduli e IA (Domain Layer, AI Gateway)
- `@marthinaf03` — Sviluppatore di dati e API (Persistenza, FastAPI)

**Durata:** 4 settimane.

---

## Aggiornamento — Organizzazione GitHub e nuove richieste del professore

Da qui in avanti la roadmap tiene conto di due informazioni nuove, arrivate dopo la stesura iniziale:

1. **L'organizzazione GitHub esiste già**: [`unibo-dtm-se-2526-bridgeit`](https://github.com/unibo-dtm-se-2526-bridgeit). Contiene (o conterrà) sia il repository `artifact` (codice) sia il repository `report` (documentazione). Il compito non è più "creare l'organizzazione", ma **migrare la documentazione già scritta in `artifact/docs/` dentro il repository `report`**, rispettandone la struttura ufficiale a 12 sezioni numerate.
2. **Il professore ha richiesto due modifiche di merito** durante la revisione della sezione Concetto: (a) includere la gestione utenti (user management) nello scope del progetto, precedentemente escluso; (b) specificare **SQLite** come tecnologia di persistenza concreta, non più generica. Queste modifiche toccano `domain-model.md`, `architecture.md` e `report.md` e vanno tracciate esplicitamente prima di essere implementate in codice.

Entrambi i punti sono dettagliati nelle due checklist qui sotto. Sono trasversali alle 4 settimane già pianificate: vanno risolti a livello di documentazione **prima o in parallelo** alla Settimana 1, così che il codice scritto nelle settimane successive parta da una documentazione già coerente.

### A. Migrazione della documentazione nel repository `report`

Il repository `report` richiede la seguente struttura a 12 sezioni numerate. Per ciascuna, indico se il contenuto esiste già (e dove) o se va scritto da zero.

- [✅] **01-concetto** —   `Concept.md` (bozza già scritta, con tipologia di prodotto e casi d'uso). Da rifinire con le due modifiche richieste dal professore (vedi checklist B) prima di considerarlo definitivo.
- [] **02-requisiti** — ✅ Contenuto in gran parte pronto: da `report.md` (Problem Statement, Domain Terminology, Project Objectives/FR/NFR, User Stories, Scope, Stakeholders). Da integrare con il nuovo FR per lo user management (vedi checklist B).
- [ ] **03-design** — ✅ Contenuto in gran parte pronto: da `architecture.md` (Architectural Drivers, Architecture, AI Architecture, Adapter Responsibilities) e `domain-model.md` (Domain Model completo). Da aggiornare con la nuova entità `User` e la scelta di SQLite (vedi checklist B).
- [ ] **04-sviluppo** — ⚠️ Parzialmente pronto: `report.md` (Development Methodology) e `architecture.md` (Proposed Package Structure) coprono i principi, ma vanno integrati con le istruzioni pratiche di setup già scritte nel README del repository `artifact` (Poetry, `poe`, comandi di test/lint).
- [ ] **05-validazione** — ✅ Contenuto pronto: `report.md` — Testing Strategy (unit/integration/acceptance, test pyramid, catena FR→US→Test Case).
- [ ] **06-release** — ✅ Contenuto pronto: `report.md` — Version Control Convention e License. Da integrare, quando disponibile, con l'esito reale della prima release via CI/CD.
- [ ] **07-dispiegamento** — ❌ Da scrivere: nessun contenuto esiste ancora. `report.md` — Current Limitations dichiara esplicitamente "no production deployment exists"; questa sezione andrà scritta solo quando (e se) un deployment reale verrà pianificato.
- [ ] **08-cicd** — ✅ Contenuto pronto: `report.md` — Continuous Integration and Continuous Delivery (pipeline GitHub Actions pianificata).
- [ ] **09-guida utente** — ❌ Da scrivere: richiede che almeno gli endpoint FastAPI siano implementati (Settimana 4). Non anticipare contenuti prima che esistano.
- [ ] **10-devguide** — ⚠️ Parzialmente pronto: il README del repository `artifact` contiene già le istruzioni di setup (Poetry, `poe test`, `poe static-checks`); vanno solo trasferite e adattate al contesto del repository `report`.
- [ ] **11-autovalutazione** — ❌ Da scrivere a fine progetto: si baserà su `report.md` — Current Limitations and Future Challenges e Conclusion, confrontando quanto pianificato con quanto realmente implementato (coerente con il task già previsto in Settimana 4).
- [ ] **12-futuro** — ✅ Contenuto pronto: `report.md` — Current Limitations and Future Challenges copre già puntualmente le direzioni di lavoro futuro; da adattare al formato della sezione.

### B. Modifiche di merito richieste dal professore (User Management + SQLite)

- [ ] **`domain-model.md`**
  - [ ] Rimuovere, in *Modeling Assumptions and Boundaries*, la frase che dichiara fuori scope l'autenticazione/user management.
  - [ ] Aggiornare la nota sull'attribuzione in `ValidationDecision`, che rimandava la questione dell'identità a "quando l'identity management verrà definito nello scope" — ora è il momento.
  - [ ] Aggiungere una nuova entità `User` (o simile) in *Domain Entities*, con un attributo di ruolo (Business Stakeholder / Requirements Engineer / Software Engineer). Decidere se è un aggregato indipendente o collegato a `ValidationDecision`.
  - [ ] Estendere il diagramma Mermaid e la tabella *Ubiquitous Language* di conseguenza.
- [ ] **`architecture.md`**
  - [ ] Aggiornare *Repository Pattern*, *Proposed Package Structure* e *Adapter Responsibilities* per nominare esplicitamente **SQLite** al posto di "tecnologia di storage non ancora decisa".
  - [ ] Valutare una seconda porta/adapter dedicata (User Repository), distinta da quella per `Requirement`.
  - [ ] Aggiungere in *API Design* gli endpoint per la gestione utenti (creazione utente, login), una volta decisi i dettagli.
- [ ] **`report.md`**
  - [ ] *Scope*: esplicitare che lo user management è ora incluso.
  - [ ] *Functional Requirements*: aggiungere almeno un nuovo FR (es. FR-08) per la gestione account/ruolo, per non lasciare la catena FR→User Story→Test Case incompleta.
  - [ ] *Technologies*: aggiungere una riga per **SQLite**.
  - [ ] *Roadmap*: aggiornare la Milestone 3 (persistenza) per riflettere SQLite fin da subito, ed eventualmente prevedere l'user management in una milestone.

---

## Mappa Settimane → Milestone (da `report.md`)

| Settimana | Milestone coperte |
|---|---|
| 1 | Inizio Milestone 2 (Domain Model) + setup FastAPI/Poetry |
| 2 | Fine Milestone 2 + inizio Milestone 3 (persistenza) e Milestone 4 (AI Gateway) |
| 3 | Fine Milestone 3 e 4 (incluso FR-05) + inizio Milestone 5 (Traceability) |
| 4 | Fine Milestone 5 + Milestone 6 (Testing e aggiornamento documentazione) |

---

## Settimana 1 — Setup e Dominio

### `@nikytresca`

- [ ] **Migrare la documentazione nel repository `report`**, sotto l'organizzazione [`unibo-dtm-se-2526-bridgeit`](https://github.com/unibo-dtm-se-2526-bridgeit), seguendo la checklist dettagliata in *Migrazione della documentazione nel repository `report`* qui sopra.
  - [ ] Verificare che sia `artifact` sia `report` siano già sotto l'organizzazione; in caso contrario, spostarli.
  - [ ] Aggiornare in `README.md` (repository `artifact`) il link `[GITHUB REPOSITORY LINK]` con l'URL definitivo del repository `report`.
  - [ ] Aggiungere `@marthinaf03` come collaboratore su entrambi i repository, se non già presente.

- [ ] **Sviluppare in Python solo l'Aggregate Root `Requirement` e il Domain Layer** (vedi `domain-model.md` — sezione Domain Entities e Aggregate Boundary).
  - [ ] Creare il package `domain/` nel repository `artifact`.
  - [ ] Definire il value object `RequirementText` (testo immutabile).
  - [ ] Definire il value object `RequirementStatus` (enumerazione: `Submitted`, `Analyzed`, `Clarified`, `Validated`, `Rejected` — vedi diagramma Mermaid in `domain-model.md`).
  - [ ] Definire l'entità `Requirement` con identità, `text`, `status`.
  - [ ] Implementare le transizioni di stato consentite come metodi dell'entità (nessuna dipendenza da framework, database o AI — solo Python puro, per rispettare l'indipendenza del Domain Layer descritta in `architecture.md`).
  - [ ] Scrivere unit test per `Requirement` (creazione, transizioni di stato valide/non valide) con `pytest`.

### `@marthinaf03`

- [ ] **Setup del progetto FastAPI base con Poetry.**
  - [ ] Verificare/estendere il progetto Poetry già inizializzato (vedi Milestone 1, già completata).
  - [ ] Aggiungere FastAPI e Uvicorn come dipendenze (`poetry add fastapi uvicorn`).
  - [ ] Creare un endpoint minimo `/health` per verificare che il server parta correttamente (`poetry run uvicorn ...`).
  - [ ] Verificare che `poetry run poe static-checks` e `poetry run poe test` continuino a funzionare dopo le modifiche.

- [ ] **Creare le interfacce (Porte) per il Database e definire i modelli Pydantic.**
  - [ ] Creare il package `application/` con un modulo per le "porte" (interfacce astratte, es. `RequirementRepository` come classe astratta/`Protocol`).
  - [ ] Definire i modelli Pydantic per la validazione delle richieste/risposte HTTP (schema di `Requirement` per l'API, distinto dall'entità di dominio — vedi `architecture.md`, Dependency Rules: l'API non deve esporre direttamente gli oggetti di dominio).
  - [ ] Scrivere un test che verifichi che la porta sia rispettata da un'implementazione fittizia (in-memory), in preparazione per Settimana 2.

---

## Settimana 2 — Motore AI e Database

### `@nikytresca`

- [ ] **Sviluppare l'Adapter per l'AI (connessione a Gemini)**, corrispondente a FR-02 (`architecture.md` — AI Architecture).
  - [ ] Creare il modulo `infrastructure/ai/` con il client Gemini.
  - [ ] Definire il port `AIGateway` (interfaccia astratta) nel Application Layer.
  - [ ] Implementare l'adapter Gemini che rispetta il port, traducendo la risposta dell'API in un oggetto di dominio `AI Analysis` (vedi `domain-model.md`).
  - [ ] Scrivere test **con un client Gemini mockato** (mai chiamare l'API reale nei test automatici, per non consumare quota e per test deterministici).

- [ ] *(Opzionale / stretch — non è ancora un requisito documentato)* Valutare un meccanismo di cache per le chiamate AI, per risparmiare token in fase di test/demo. Se implementato, **aggiungere prima una riga in `report.md` (nuovo NFR o nota tecnica)** per documentare la decisione, coerentemente con la filosofia del progetto.

### `@marthinaf03`

- [ ] **Sviluppare l'Adapter per SQLite utilizzando il pattern Repository** (vedi `architecture.md` — Repository Pattern, Adapter Responsibilities).
  - [ ] Creare il modulo `infrastructure/persistence/`.
  - [ ] Implementare `SQLiteRequirementRepository` conforme alla porta definita in Settimana 1.
  - [ ] Gestire la connessione al database (file locale, coerente con l'approccio già usato nel bootstrap iniziale del backend).

- [ ] **Scrivere i test per verificare il salvataggio su database locale.**
  - [ ] Test di salvataggio e recupero di un `Requirement` tramite il repository.
  - [ ] Test che verifica che il repository non alteri il contenuto dell'entità (nessuna perdita di dati tra scrittura e lettura).

---

## Settimana 3 — Casi d'uso e Integrazione

### `@nikytresca`

- [ ] **Sviluppare gli Use Cases (Application Layer)** che collegano Dominio, persistenza e AI. Ogni caso d'uso corrisponde a un FR già documentato:
  - [ ] Caso d'uso "Sottometti requirement" (FR-01).
  - [ ] Caso d'uso "Richiedi analisi AI" (FR-02).
  - [ ] Caso d'uso "Chiarisci requirement" (FR-03).
  - [ ] Caso d'uso "Ottieni Quality Indication" (FR-04).

- [ ] **FR-05 — Validazione Umana dei suggerimenti AI (fondamentale, non presente nella bozza originale).**
  - [ ] Implementare il caso d'uso che registra una `ValidationDecision` (approva / modifica / rifiuta) su una `AI Analysis` in attesa.
  - [ ] Garantire che nessuna `AI Analysis` possa modificare lo stato autoritativo del `Requirement` senza una decisione umana esplicita registrata (vedi `domain-model.md` — Aggregate Boundary).
  - [ ] Scrivere test che verificano esplicitamente che, senza decisione umana, lo stato del requirement non cambi.

- [ ] **Scrivere i test di logica aziendale**, puntando a un'alta copertura del Domain Layer e dell'Application Layer (vedi `report.md` — Testing Strategy, principio della test pyramid).

### `@marthinaf03`

- [ ] **Creare le rotte FastAPI (Endpoints)** che richiamano gli Use Cases, riprendendo esattamente gli endpoint già definiti in `architecture.md` — API Design:
  - [ ] `POST /requirements` (FR-01)
  - [ ] `GET /requirements/{id}`
  - [ ] `POST /requirements/{id}/analyse` (FR-02)
  - [ ] `POST /requirements/{id}/validate` (FR-05)
  - [ ] *(se il tempo lo consente)* `GET /requirements/{id}/traceability-links` (FR-06) e `POST /requirements/{id}/artifacts` (FR-07), come inizio della Milestone 5.

- [ ] **Verificare il funzionamento completo tramite Swagger UI** (`/docs`, generato automaticamente da FastAPI).
  - [ ] Testare manualmente il flusso completo: crea requirement → richiedi analisi → valida → (se implementato) crea artefatto.
  - [ ] Segnalare a `@nikytresca` eventuali incoerenze tra il comportamento reale e quanto descritto in `architecture.md`.

---

## Settimana 4 — Report e Rifiniture Finali

### `@nikytresca`

- [ ] **Aggiornare (non riscrivere) `architecture.md` e `domain-model.md`** per riflettere eventuali scostamenti tra quanto pianificato e quanto effettivamente implementato. I due documenti sono già completi nella loro versione "pianificata": il lavoro qui è verificare che ogni sezione sia ancora vera, non produrne di nuove.
- [ ] **Revisione finale della CI/CD (GitHub Actions)**, secondo quanto già descritto in `report.md` — Continuous Integration and Continuous Delivery: pipeline con install dipendenze, `poetry run poe static-checks`, `poetry run poe test`.

### `@marthinaf03`

- [ ] **Aggiornare la sezione di `report.md` relativa a test e persistenza** (Testing Strategy, Current Development Status), sostituendo le parti descritte come "pianificate" con lo stato reale una volta implementate.

### `@nikytresca` e `@marthinaf03` (insieme)

- [ ] **Test end-to-end del prototipo completo** prima della consegna: percorso Requirement → AI Analysis → Human Validation → Traceability Link → Derived Artifact, eseguito manualmente e, se possibile, come test di accettazione automatizzato.
- [ ] **Aggiornare `report.md` — "Current Limitations and Future Challenges" e "Conclusion"** con lo stato reale raggiunto a fine progetto, senza dichiarare implementato nulla che non lo sia davvero.
- [ ] Verificare che tutti i link incrociati tra `report.md`, `architecture.md` e `domain-model.md` (e verso il repository `report`) siano ancora validi dopo le modifiche.