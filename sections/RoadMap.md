# Roadmap del Progetto e Assegnazione Task

Questa roadmap definisce gli obiettivi settimanali per il team e **opera concretamente le Milestone già definite in `report.md` (sezione Roadmap)**: non è un piano alternativo, ma la traduzione di quelle milestone in task settimanali assegnati a persona, con sotto-task a livello principiante.

**Squadra:**
- `@nikytresca` — Sviluppatrice di moduli e IA (Domain Layer, AI Gateway)
- `@marthinaf03` — Sviluppatore di dati e API (Persistenza, FastAPI)

**Durata:** 4 settimane.

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

- [ ] **Creare l'Organizzazione GitHub e trasferire i repository**, coerentemente con la sezione "Repository Organization" del `README.md`.
  - [ ] Creare l'organizzazione GitHub condivisa dal team.
  - [ ] Spostare/collegare il repository `artifact` (codice) e il repository `report` (già esistente: https://github.com/nikytresca-pixel/report) sotto l'organizzazione.
  - [ ] Aggiornare in `README.md` il link `[GITHUB REPOSITORY LINK]` con l'URL definitivo.
  - [ ] Aggiungere `@marthinaf03` come collaboratore su entrambi i repository.

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