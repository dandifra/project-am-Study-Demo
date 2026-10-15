# ⚠️ DOCUMENTO INTERNO FORNITORE — NON CONDIVIDERE CON IL CLIENTE

> 3. Pipeline CI/CD matura per iterazione test-driven
> 4. Rapporto senior/junior ≥ 2:1 nel team delivery
> 5. Cultura di prompting esplicita e pattern di contesto documentati
>
> Se il team non soddisfa questi prerequisiti, il fornitore assorbe internamente il rischio di fallback; il cliente resta garantito al prezzo di offerta.

| Voce | Valore |
|------|--------|
| **Classificazione** | CR su progetto esistente (Enhancement RPA-10) |
| **Effort baseline** (rif. interno) | **€ 5.040** (range €4.200–€5.880) — delivery tradizionale |
| **Effort offerta** (agentico) | **€ 3.150** (12 gg sviluppo + 3 gg preparazione ambiente — range €2.625–€3.675) |
| **Buffer GenAI** (rischio interno) | **€ 1.890** — assorbe eventuale fallback tooling |
| **Costo interno** (15 gg × 210 €/gg) | **€ 3.150** |
| **Ricavo** (Costo interno +25%) | **€ 3.937,50** |
| **Margine atteso** | **€ 787,50 (+25%)** |
| **Durata stimata** | ~5,5 settimane (2,5 sett. sviluppo + 3 sett. staffing — staffing non incluso nel costo) |
| **Team mix** | Senior €1.785 (8,5 gg) + Mid €735 (3,5 gg) |

| **🚨 Rischio #1** | Tecnico: struttura fogli SCTR per RETELIT non ancora disponibile — RETELIT completamente assente dal codebase (confermato dalla lettura al 100% del codice) |
| **Raccomandazione PM** | 1. Ottenere almeno 1 file SCTR di test per RETELIT prima dell'avvio sviluppo. 2. ✅ Codice sorgente acquisito e letto al 100% — R3 chiuso. 3. Pianificare milestone a 7 gg dall'avvio. 4. Non fare sviluppo RETELIT cieco senza file SCTR di test. |

---

## 1. Classificazione e razionale

**Categoria:** CR su progetto esistente (Enhancement)

**Razionale:**
- L'AdF §REQ001 richiede esplicitamente l'**adeguamento** di un sistema esistente (RPA-10_Quote) già in produzione presso TIM
- Il perimetro è ben delimitato: modifiche (CHG) e aggiunte (ADD) a logiche di elaborazione report già strutturate
- Il sistema ha già una base di codice funzionante per operatori analoghi (POSTEPAY FULL MVNO, Vodafone, Kena/Noverca); si tratta di estendere pattern esistenti a nuovi operatori
- I dati di test sono già disponibili (`REP_SCTR_202603/`: 18 file XLS reali del ciclo marzo 2026) — riduce il rischio di regression testing
- Il collaudo utente è pianificato in 3 gg standard (§9 AdF) e non è bloccante per la classifica CR

---

## 2. Classificazione e razionale

*(vedi §1 sopra)*

---

## 3. Sintesi di cosa è stato letto

### 3.1 Documento AdF

- **Codice progetto PPM**: 373443
- **Titolo**: RPA 10 Quote SCTR Adeguamento nuovi operatori (v1.1 — 05/05/2026)
- **Obiettivi di business**: adeguare l'automa RPA-10_Quote per acquisire e trasformare i report SCTR dei nuovi operatori MVNO (POSTEPAY, Lyca, RETELIT) ai fini del processo "Quote da riversare per le competenze del mobile" gestito da LU CF.FRA.R
- **Vincoli / scadenze**: POSTEPAY con carattere d'urgenza; Lyca e RETELIT "in concomitanza con adeguamenti SCTR" (timing dipendente da SCTR)
- **Clientela target**: LU CF.FRA.R — Accounting & Revenue Assurance
- **Perimetro funzionale**: modifiche al sistema ROBOTIC-PROCESS-AUTOMATION, modulo Quote SCTR. Impatti su Entrante, Uscente, Internazionale, SMS Uscente
- **Tecnologie citate**: ROBOTIC-PROCESS-AUTOMATION (implicito: Python/openpyxl o equivalente), SCTR, file .xls
- **Contratto**: K24AAGLF9W — ENGINEERING INGEGNERIA INFORMATICA
- **Sicurezza**: tutte le domande di sicurezza §6.1 risultano NO — nessuna NT.1 applicabile

### 3.2 Documentazione analizzata

| File | Tipo | Stato | Note |
|------|------|-------|------|
| `AdF- 373443 RPA 10 Quote SCTR Adeguamento nuovi operatori v1.1.docx` | AdF (Word) | Letto integralmente | 15 sezioni, 2 revisioni (v1.0/v1.1) |
| `Report Svil_FFPA_UdA V5.0- SCTR_Integrazione nuovi operatori.xlsm` | Analisi funzionale pre-esistente | Letto integralmente | Analisi precedente su 3 operatori (Lyca, RETELIT, POSTEPAY) |
| `REP_SCTR_202603/` (18 file XLS) | Report SCTR marzo 2026 | Inventario completo | Dati reali di produzione: DECADE1, ENTRANTE, INTERNAZIONALE, SMS_*, NNG_*, NUMERI_VERDI, USCENTE, Videocall_*, MMS_* |

### 3.3 Codice sorgente analizzato

Il codice sorgente del sistema RPA-10 è stato **letto al 100%** tramite due subagent paralleli + lettura diretta.

**sctr-BE** (backend del robot SCTR):
- **49 file C#, 8.242 LOC** — Worker Service .NET con EPPlus (Excel), Entity Framework Core, DB SQL Server
- Stack confermato: **C# .NET** (non Python come originalmente ipotizzato)
- Architettura: `BotTrasformation` → dispatch per prefix file → classi dedicate in `Trasformazioni/Files/` (Entrante, Uscente, Internazionale, SMS_Uscente, ecc.) + `Trasformazioni/Report/` (file di quadratura)
- **FINDING CRITICO 1**: LYCA (LYCAMOBILE FULL MVNO) è già implementato in `Uscente.cs` (metodo `LYCAMOBILE_FULL_MVNO`, riga 970), `SMS_Uscente.cs` (metodo `SMS_TIM_LYCAMOBILE_FULL_MVNO`, riga 244), `UscenteVSAOM.cs` (fogli LYCAMOBILE). → **P09 Lyca eliminato dalla stima**
- **FINDING CRITICO 2**: POSTEPAY (POSTEPAY FULL MVNO) è già implementato in `Uscente.cs` (riga 777), `SMS_Uscente.cs`, `UscenteVSAOM.cs` (fogli POSTEMOBILE). → **P03 ridotto**
- **FINDING CRITICO 3**: RETELIT è **completamente assente** dall'intero codebase (0 occorrenze su 49 file). → **P10 RETELIT confermato ADD**
- Internazionale.cs: nessun handler per POSTEPAY, LYCA, RETELIT (solo Noverca/Kena, T-Mobile, fogli fisso/mobile). → P05 confermato
- `Entrante.cs`: nessun sheet "POSTE MVNO". → P02 confermato
- `SMS_Uscente.cs`: nessun sheet "SMS_TIM_COOP". → P07 confermato

**sctr-FE** (portale di configurazione):
- **395 file C#, 14.792 LOC** — Blazor .NET + ASP.NET Core WebAPI
- Il portale gestisce già CRUD completo per Anagrafica operatori (`AnagraficaController.cs` con GET/POST/PUT/DELETE/Export)
- La configurazione dei nuovi operatori (RETELIT) avviene tramite il portale esistente — **nessuna modifica al codice del portale richiesta**
- Nessuna occorrenza di RETELIT nel portale (attesa — è configurazione dati)

---

## 4. Breakdown effort baseline

Il baseline è la stima **senza** uso di GenAI — riferimento teorico interno.

| Fase | Min (€) | Max (€) | Valore puntuale (€) | Note |
|------|----------|----------|----------------------|------|
| Analisi AdF + studio codice SCTR | €210 | €420 | €315 | Codice disponibile e letto: analisi più rapida e precisa |
| Design tecnico (mapping modifiche) | €168 | €315 | €210 | Pattern già presente nel codebase (LYCA → adattare per RETELIT) |
| Sviluppo (9 processi funzionali) | €1.785 | €2.730 | €2.205 | Dettaglio per processo in §4.1 — P09 Lyca eliminato |
| Unit test (con file REP_SCTR_202603 reali) | €525 | €840 | €630 | Dati reali disponibili; test ripetibili con xUnit/.NET |
| Integration test / end-to-end | €210 | €420 | €315 | Simulazione ciclo SCTR completo |
| UAT support (CU 3gg previsto in AdF §9) | €210 | €420 | €315 | Presenza tecnica a supporto LU |
| Deployment applicativo RPA-10 | €105 | €210 | €105 | Deploy Worker Service su ambiente produzione |
| Documentazione tecnica | €105 | €210 | €105 | Aggiornamento changelog e configurazioni |
| Project management | €105 | €315 | €210 | Coordinamento, reporting, allineamenti |
| Buffer / contingency (15%) | €525 | €840 | €630 | Buffer ridotto: codice disponibile abbassa rischio strutturale |
| **TOTALE baseline** | **€3.990** | **€6.720** | **€5.040** | Confidenza: **alta** (codice letto al 100%) |

### 4.1 Breakdown sviluppo per processo funzionale (baseline)

| Processo | Flag | € baseline | Note |
|----------|------|------------|------|
| P1 - Configurazione anagrafica RETELIT (via portale esistente) | CHG | €105 | Inserimento record DB via portale già CRUD-ready; nessun code change portale |
| P2 - Entrante: aggiornamento acquisizione (POSTE MVNO) | CHG | €210 | Nuovo sheet + rimosso sheet MIGRAZIONE POSTE MVNO; pattern da codice esistente |
| P3 - Uscente: POSTE MVNO (POSTEPAY già implementato) | CHG | €315 | Solo sheet POSTE MVNO e inferenza dedicata; POSTEPAY FULL MVNO già in produzione |
| P4 - Produzione quadratura Uscente (POSTE MVNO + riepilogo) | ADD | €315 | 2 nuovi output: foglio + file riepilogo; pattern da codice esistente |
| P5 - Internazionale: acquisizione + trasformazione (POSTEPAY+LYCA+RETELIT) | CHG | €420 | 3 nuovi operatori per Internazionale (tutti assenti dal codice) |
| P6 - Produzione quadratura Internazionale (riepilogo POSTE) | ADD | €168 | Nuovo file riepilogo scenari POSTE; pattern da altri Report |
| P7 - SMS Uscente: aggiornamento + trasformazione (SMS_TIM_COOP) | CHG | €168 | Nuovo sheet SMS_TIM_COOP; pattern da codice esistente |
| P8 - Produzione quadratura SMS (SMS_TIM_COOP) | ADD | €84 | Nuovo foglio output; struttura semplice |
| P10 - Elaborazione SCTR RETELIT (Uscente+SMS+Quadratura) | ADD | €420 | RETELIT assente dal codebase; pattern LYCA già in codice come riferimento |
| **TOTALE sviluppo** | | **€2.205** | P09 Lyca eliminato: già implementato in produzione |

---

## 5. Efficienza GenAI & Agentic

> **⚠ Questa stima assume delivery full-agentic.** Il team esegue il lavoro usando AI agentica impegnata (Claude Code / Cursor Agent) come paradigma primario, NON come autocompletamento IDE. Prerequisiti impliciti a carico fornitore:
> 1. Senior developer culturati sulla supervisione di output agentici
> 2. Tooling agentico installato e funzionante (licenze non incluse nel prezzo cliente)
> 3. Pipeline CI/CD matura per iterazione test-driven
> 4. Rapporto senior/junior ≥ 2:1 nel team delivery
> 5. Cultura di prompting esplicita e pattern di contesto documentati
>
> Se il team non soddisfa questi prerequisiti, il fornitore assorbe internamente il rischio di fallback; il cliente resta garantito al prezzo di offerta.

### 5.1 Workflow agentici concreti per questo progetto

1. **Agente analisi struttura codice C#**: Claude Code legge Uscente.cs (1.247 righe) e produce automaticamente il template per il nuovo metodo `RETELIT_FULL_MVNO()` — copy-adapt del pattern `LYCAMOBILE_FULL_MVNO()` già presente alla riga 970 → riduzione −65% coding RETELIT in Uscente
2. **Generazione transformer Internazionale per POSTEPAY/LYCA/RETELIT**: Claude Code legge Internazionale.cs (struttura semplice, 8 handler esistenti) e genera i 3 nuovi handler per operatori MVNO per pattern replication → riduzione −60% coding Internazionale
3. **Generazione handler SMS e Report**: Claude Code genera `SMS_TIM_RETELIT_FULL_MVNO()` da pattern `SMS_TIM_LYCAMOBILE_FULL_MVNO()` e il blocco RETELIT in UscenteVSAOM.cs con aggiornamento formule SUM → riduzione −65% coding SMS + Report
4. **Test unit con xUnit**: Claude Code genera test xUnit (o MSTest) leggendo file REP_SCTR_202603 reali, applicando le trasformazioni e confrontando output — riduzione −65% unit testing
5. **Produzione file quadratura nuovi fogli**: Claude Code genera le funzioni EPPlus per i nuovi fogli (POSTE MVNO, riepilogo POSTE vs AOM, SMS_TIM_COOP) per pattern replication da UscenteVSAOM.cs esistente → riduzione −55% coding output

### 5.2 Uplift per fase

| Fase | Uplift % | Motivazione (ancorata al progetto) |
|------|----------|------------------------------------|
| Analisi AdF + studio codice | −45% | Codice disponibile e già letto; pattern di ogni handler identificati; analisi di impatto precisa |
| Design tecnico | −40% | Pattern LYCA in Uscente.cs è il template esatto per RETELIT — design quasi automatico |
| Sviluppo (9 processi) | **−62%** | Stack C# .NET mainstream — eccellente training data per Claude Code. Pattern ripetibile (RETELIT ≈ LYCA pattern già nel codebase). Internazionale ha struttura semplice (3 handler simili). |
| Unit test | **−65%** | File XLS reali disponibili in REP_SCTR_202603. AI genera test xUnit comparando output pre/post con dati reali — caso ideale per test agentico |
| Integration test | −40% | Ciclo SCTR (timing 3 del mese) richiede coordinamento umano; test script C# generabili |
| UAT support | **0%** | Presenza tecnica durante CU di 3 gg previsto da AdF: attività irriducibile umana |
| Deployment applicativo | −40% | Script di deploy generabili; go/no-go è umano |
| Documentazione | −55% | AI genera changelog tecnico e aggiornamento configurazioni in C# |
| Project management | **0%** | Coordinamento LU CF.FRA.R, allineamenti SCTR: irriducibile umano |

### 5.3 Assunzioni e paletti obbligatori

- **Seniority minima**: Senior Developer con esperienza C#/.NET + utilizzo Claude Code come agente (non autocomplete). Lo stack C# EPPlus è mainstream ma richiede conoscenza delle API EPPlus per la manipolazione fogli Excel
- **Quality gate umani obbligatori**: revisione sistematica di ogni handler generato su file REP_SCTR_202603 reali; zero shipping di output agentico senza esecuzione test suite xUnit
- **Tool assunti**: Claude Code (o Cursor Agent) con accesso completo al codice sctr-BE; file REP_SCTR_202603 disponibili nella sessione agentica come golden dataset
- **Rischio allucinazione**: concentrato sulle regole di business specifiche (esclusione Kena/Noverca da sheet specifici, formule SUM in UscenteVSAOM.cs che elencano tutti gli operatori) — richiedono revisione umana obbligatoria
- **Costi licenze NON inclusi**: Claude Code / Cursor Agent / Cline — a carico ENGINEERING, non nel prezzo cliente
- **Vincolo timing RETELIT**: i file SCTR per RETELIT non sono ancora disponibili (0 occorrenze nel codebase confermano che RETELIT non è ancora in produzione). Il rischio è sviluppare su struttura ipotetica → prevedere revisione a 7 gg dall'avvio

### 5.4 Attività NON beneficianti di GenAI

- **UAT support** (1,5 gg): presenza fisica durante il CU — attività umana, non delegabile
- **Project management** (1 gg): allineamenti LU CF.FRA.R, coordinamento con SCTR per timing RETELIT — relazionale e non tecnico
- **Verifica business rules critiche**: la regola "escludere Kena/Noverca dal totale" ha impatto diretto sulla fatturazione TIM/operatori — revisione umana obbligatoria prima di ogni commit; in particolare per le formule SUM in UscenteVSAOM.cs che devono includere il nuovo sheet RETELIT

---

## 6. Breakdown effort GenAI-accelerated (scenario di offerta)

| Fase | € baseline | Uplift % | € agentico | Delta (€) |
|------|-------------|----------|------------|------------|
| Analisi AdF + studio codice | €315 | −45% | €168 | −€147 |
| Design tecnico | €210 | −40% | €126 | −€84 |
| Sviluppo (9 processi) | €2.205 | −62% | €840 | −€1.365 |
| Unit test | €630 | −65% | €210 | −€420 |
| Integration test | €315 | −40% | €189 | −€126 |
| UAT support | €315 | 0% | €315 | €0 |
| Deployment applicativo | €105 | −40% | €63 | −€42 |
| Documentazione | €105 | −55% | €42 | −€63 |
| Project management | €210 | 0% | €210 | €0 |
| Buffer / contingency (10%) | €630 | — | €210 | −€420 |
| **Preparazione ambiente di sviluppo** | — | — | **€630** | **3 gg** × €210 — aggiuntivi, non upliftabili da GenAI |
| **TOTALE** | **€5.040** | **−37%** | **€3.150** | **−€1.890** |

**Range agentico**: €2.625–€3.675 (inclusi 3 gg prep ambiente) | **Uplift medio sviluppo**: −50%

---

## 7. Team mix proposto — versione iniziale

Dimensionato sull'effort agentico (15 gg: 12 gg sviluppo + 3 gg preparazione ambiente). Pattern A — CR su esistente (da `team-mix-patterns.md`).

| Ruolo | € allocati | % | Motivazione |
|-------|------------|---|-------------|
| Project Manager / Delivery Lead | €315 | 10,0% | Coordinamento LU CF.FRA.R, allineamento SCTR per RETELIT, CU gestione |
| Architect / Tech Lead (Senior) | €315 | 10,0% | Valutazione impatto su Worker Service, revisione output agentico su regole di business critiche |
| Senior Developer (agentic lead) | €1.890 | 60,0% | Execution Claude Code/Cursor Agent su sctr-BE; implementazione RETELIT e nuovi operatori; preparazione ambiente di sviluppo (3 gg) |
| QA / Test Engineer (Mid) | €525 | 16,7% | Test con file REP_SCTR_202603, verifica output quadratura, supporto UAT |
| DevOps (pipeline applicativa) | €105 | 3,3% | Deploy Worker Service su ambiente produzione |
| **TOTALE** | **€3.150** | **100%** | Ratio senior/junior: **≥ 2:1 ✓** |

**Durata calendar**: 15 gg / (2 FTE × 20 × 0,8) ≈ **0,47 mesi ≈ 2,5 settimane** sviluppo + **15 gg staffing** (non incluso nel costo) = **~5,5 settimane totali elapsed**

---

## 8. Domanda nearshore e team mix ibrido

### 8.1 Domanda posta all'utente
> **Vuoi valutare l'opzione nearshore (es. Est Europa, Balcani, altre sedi del gruppo) per parte del team?**

### 8.2 Risposta dell'utente
**Onshore only** — nearshore non richiesto.

---

## 9. Assunzioni generali

- Stack target **confermato**: **C# .NET Worker Service con EPPlus** per manipolazione Excel, Entity Framework Core + SQL Server per anagrafica
- Codice sorgente RPA-10 letto al 100%: sctr-BE (49 file, 8.242 LOC) + sctr-FE (395 file, 14.792 LOC)
- File di test reali disponibili: REP_SCTR_202603/ (18 file XLS) — dati ciclo marzo 2026
- Ambienti: ambiente sviluppo + ambiente collaudo utente (CU 3 gg) + produzione
- Il timing di sviluppo per RETELIT dipende dal rilascio SCTR — il team non può testare senza file SCTR reali di RETELIT (LYCA è già disponibile: operatore già in produzione nel codebase)
- Calendario standard: 8 ore/giornata, esclusi weekend e festività italiane
- Costo interno blended confermato dall'utente: **210 €/gg**

---

## 10. Rischi e contingency

| # | Rischio | Severità | Probabilità | Impatto | Mitigazione |
|---|---------|----------|-------------|---------|-------------|
| R1 | File SCTR per RETELIT non disponibili al momento dello sviluppo — struttura fogli sconosciuta | ALTA | ALTA | +2–4 gg rework se struttura diversa da attesa | Concordare con SCTR il timing rilascio e ottenere file campione PRIMA di avviare sviluppo per RETELIT; LYCA è già in produzione — nessun rischio per il pattern di riferimento |
| R2 | Regole di business su esclusione Kena/Noverca malinterpretate (impatto su fatturazione) | ALTA | MEDIA | Errore silente nella quadratura = errata determinazione quote | Revisione umana obbligatoria su ogni handler; test con dati reali vs valori attesi condivisi dalla LU; particolare attenzione alle formule SUM in UscenteVSAOM.cs |
| R3 | ~~Codice RPA-10 non disponibile — stack non confermato~~ | ~~MEDIA~~ | ~~MEDIA~~ | **✅ CHIUSO**: codice letto al 100%, stack C# confermato | — |
| R4 | Scope creep durante CU: LU chiede modifiche non previste in AdF | MEDIA | MEDIA | +1–3 gg | Definire in AdF che variazioni al CU sono CR separate |
| R5 | Fallback da agentico a tradizionale (tooling non funzionante, team non maturo) | ALTA | BASSA | Effort sale da 15 a 23–27 gg — rischio interno fornitore | Verificare tooling prima dell'avvio; milestone a 7 gg per early-warning |
| R6 | **Magic number `== 9` in `BotTrasformationReport.cs` riga 128** — hardcoded numero file elaborati prima di trigger email | ALTA | ALTA | Email di completamento non inviata dopo aggiunta RETELIT (conta 9 ma ce ne saranno 10+) → silent failure del ciclo report | **Aggiornare il magic number** come parte del P10 RETELIT (in scope — change minore ma critico); verificare anche `BotCheck.cs` riga 29 per path hardcoded `C:\TIM\SCTR\` |
| R7 | Zero test automatizzati nel codebase — nessuna regression suite | MEDIA | CERTA | Ogni modifica è cieca; regressions silenziose sugli operatori esistenti (LYCA, POSTEPAY, Kena) non rilevate | Fortemente raccomandato creare test xUnit come ampliamento permanente (vedi §14B) |

---

## 11. Out of Scope

La stima **NON include**:

- Provisioning server/VM per RPA-10 — attività sistemistica (IT Operations)
- Configurazione OS, patching, backup piattaforma RPA — fuori perimetro skill
- Modifiche al sistema SCTR o ai suoi report (è il produttore dei file; i requisiti SCTR sono su altri PPM: 349193, 355807, 357883, 368065, 365874)
- Sviluppo lato altri sistemi che interagiscono con RPA-10 (SCTR, sistemi di fatturazione downstream)
- Formazione utenti LU CF.FRA.R — non citata in AdF
- Supporto post-produzione / maintenance — CR separata
- Attività di sicurezza tecnica (cifratura, hardening) — tutte le domande AdF §6.1 sono NO
- Backfill dati storici per nuovi operatori — non richiesto in AdF

---

## 12. Domande aperte / informazioni mancanti

- [x] ~~Accesso al repository codice sorgente RPA-10~~ ✅ **CHIUSO** — codice letto al 100%; stack C# .NET confermato; LYCA/POSTEPAY già in produzione; RETELIT assente
- [x] ~~Conferma stack tecnologico~~ ✅ **CHIUSO** — C# .NET Worker Service + EPPlus + EF Core + SQL Server
- [ ] Timing previsto per rilascio SCTR dei file RETELIT (l'AdF dice "in concomitanza con adeguamenti SCTR" — quando? LYCA non è più un rischio: già in produzione)
- [ ] Disponibilità file SCTR di test per RETELIT (1 file campione) prima dell'avvio sviluppo
- [ ] Costo interno effettivo del team assegnato (il 210 €/gg è il valore blended confermato dall'utente)

---


## 14. 📎 ANNEX — Ampliamenti tecnici proposti (NON inclusi nella stima principale)

| # | Ampliamento | Giustificazione tecnica | Effort delta (gg) | Raccomandazione |
|---|------------|--------------------------|-------------------|-----------------|
| A | ~~**Accesso al codice RPA-10 e code review preventiva**~~ | **✅ CHIUSO** — codice letto al 100% nella sessione corrente | 0 | Completato |
| B | **Test automatici di regressione permanenti** (xUnit/.NET) | Zero test automatizzati nel codebase (R8 confermato dalla lettura al 100%). La CR introduce modifiche in 4 file core (Uscente.cs, SMS_Uscente.cs, Internazionale.cs, UscenteVSAOM.cs). Un test suite permanente protegge da future regressioni in CR successive | +1–2 gg | **Must** — debito tecnico critico; ogni CR futura è a rischio senza test |
| C | **Monitoraggio automatico struttura fogli SCTR** | Il sistema potrebbe avvisare automaticamente (email/log) se la struttura dei fogli SCTR cambia inaspettatamente — prevenendo crash silenti | +1,5 gg | **Nice to have** — particolarmente utile per RETELIT che non è ancora in produzione |
| D | **Fix magic number `== 9` → dynamic count** in `BotTrasformationReport.cs` riga 128 | Attualmente la mail di completamento viene inviata quando esattamente 9 file sono elaborati. Con RETELIT il conto sarà 10+. Questo è un bug implicito che DEVE essere corretto come parte della CR | +0,3 gg | **Must** — incluso nel P10 RETELIT (già in scope, costo ≤0,5 gg) |

### 14.2 Effort delta con ampliamenti

| Scenario | gg baseline | gg GenAI | € Costo | € Ricavo (+25%) |
|----------|-------------|----------|---------|----------------|
| STRICT (come richiesto in AdF) | 24 | 12 | €2.520 | €3.150 |
| STRICT + B (test xUnit suite) | 26 | 13,5 | €2.835 | €3.544 |
| STRICT + B + C (monitoring) | 27,5 | 15 | €3.150 | €3.938 |
| STRICT + tutti | 27,8 | 15,3 | €3.213 | €4.016 |

### 14.3 Raccomandazione

Scope **STRICT + B** raccomandato (test xUnit: è un debito tecnico critico confermato dalla lettura del codice — zero test nel codebase). Ampliamento **D** (magic number fix) è già incluso in P10 RETELIT come in-scope. **C** è a discrezione della LU.

---

## ✅ Checklist pre-firma (obbligatoria per il PM)

**Gate economici**
- [x] Costo interno medio blended team confermato: 210 €/gg ✓
- [x] Margine atteso: ✅ +25% (€787,50) — Soddisfatto a 15 gg (12 gg sviluppo + 3 gg prep ambiente)
- [ ] Modello di pricing a corpo concordato con cliente

**Gate team**
- [ ] **Senior Developer con esperienza C#/.NET EPPlus + Claude Code** identificato e disponibile
- [ ] Cultura agentica verificata: Claude Code / Cursor Agent ✓ (confermato dall'utente)
- [ ] **Tooling agentico operativo con accesso al codice sctr-BE** (repository completo nella sessione agentica)
- [ ] Rapporto senior/junior nel team: 2,2:1 ✓

**Gate tecnici**
- [x] ~~Accesso al repository codice sorgente RPA-10 (stack confirmation)~~ ✅ **CHIUSO** — sctr-BE + sctr-FE letti al 100%
- [ ] File SCTR di test per **RETELIT** disponibili prima dello sviluppo (LYCA non più bloccante: già in produzione)
- [ ] Ambiente di collaudo utente disponibile per CU 3 gg (AdF §9)
- [ ] Shaping tecnico v.definitiva: AdF v1.1 del 05/05/2026 ← ultima versione disponibile ✓

**Gate processuali**
- [ ] Milestone a 7 gg dall'avvio: revisione codice agentico + primo test con REP_SCTR_202603
- [ ] Piano di fallback agentico→tradizionale definito internamente
- [ ] Scope creep rules concordate: variazioni al CU gestite come CR separate
- [ ] Timing **RETELIT** concordato con PM SCTR (LYCA non più bloccante — già in produzione nel codebase)

---

*Documento generato con la skill `it-project-estimator` — aggiornato 2026-05-15 con analisi codice sorgente al 100% (sctr-BE 49 file + sctr-FE 395 file).*
