# Simulatore Netto in Busta 2026
## Documentazione tecnica, funzionale e logica di calcolo

## 1. Obiettivo del progetto

Il progetto nasce dal task **“Product Builder @ Jet HR”**, il cui obiettivo è costruire un prototipo funzionante capace di simulare la proiezione della retribuzione netta annuale e mensile partendo dalla RAL (Retribuzione Annua Lorda).

Il prototipo deve:

- ricevere in input una RAL;
- considerare la posizione geografica del dipendente;
- calcolare i principali elementi che vengono sottratti dalla retribuzione lorda;
- mostrare il netto annuale;
- mostrare il netto medio mensile;
- mostrare separatamente contributi, IRPEF, addizionale regionale, addizionale comunale e detrazione da lavoro dipendente;
- rendere comprensibile all’utente come si passa dal lordo al netto.

Il dominio payroll italiano è molto più complesso di quanto possa essere rappresentato da un prototipo. Per questo motivo la soluzione non vuole simulare un cedolino reale in tutti i suoi dettagli, ma costruire un **modello semplificato, trasparente e funzionante**, esplicitando le principali assunzioni.

L’implementazione è contenuta in un unico file HTML che include struttura della pagina, stile CSS, dati e logica JavaScript. La pagina funziona lato client e non richiede un backend per il calcolo.

Il risultato finale è stato inoltre pubblicato online come pagina web funzionante, così da rendere il prototipo immediatamente accessibile e verificabile.

**Demo online:** https://unique-puffpuff-195472.netlify.app

---

# 2. Approccio progettuale

La scelta principale è stata quella di realizzare una **single-page application completamente client-side**, evitando per il prototipo la necessità di:

- backend;
- database;
- API proprietarie;
- gestione di account;
- persistenza dei dati dell’utente.

La pagina dichiara esplicitamente che i dati inseriti rimangono nel browser e che il modulo non viene inviato a un server.

Questa scelta è coerente con l'obiettivo del test: realizzare rapidamente un prototipo funzionante, mantenendo però le logiche di business leggibili e modificabili.

Il file è quindi organizzato in quattro macro-strati:

1. **Presentazione e UI**
   - HTML;
   - CSS;
   - responsive design.

2. **Dataset**
   - aliquote regionali;
   - aliquote comunali;
   - elenco dei Comuni;
   - informazioni CCNL.

3. **Motore di calcolo**
   - contributi;
   - imponibile fiscale;
   - IRPEF;
   - detrazione;
   - addizionale regionale;
   - addizionale comunale;
   - netto annuale e mensile.

4. **Visualizzazioni aggiuntive**
   - breakdown del netto;
   - indicatore delle trattenute;
   - mappa regionale;
   - tabella comparativa tra Regioni.

---

# 3. Struttura della pagina

La pagina è composta da una serie di sezioni funzionali.

## 3.1 Hero

La parte iniziale introduce il prodotto e spiega in maniera sintetica il suo funzionamento.

Viene inoltre spiegato che il simulatore considera:

- RAL;
- Regione;
- Comune;
- numero di mensilità;
- contributi;
- IRPEF;
- addizionali;
- detrazione da lavoro dipendente.

La pagina chiarisce fin dall'inizio che si tratta di una **stima** e non di un calcolo payroll definitivo.

---

# 4. Input del simulatore

Il calcolatore principale riceve quattro informazioni:

## 4.1 RAL

L'utente inserisce la propria RAL annuale lorda.

Il valore iniziale preimpostato è €40.000.

La RAL rappresenta il punto di partenza dell'intero modello.

---

## 4.2 Regione di residenza

La Regione viene selezionata tramite una select contenente le 20 Regioni italiane.

La Regione è necessaria perché l'addizionale regionale IRPEF varia in funzione della Regione di residenza fiscale.

Il codice contiene un dataset denominato:

```javascript
REGIONAL_RATES_2026
```

che associa ogni Regione ai propri scaglioni e alle relative aliquote.

---

## 4.3 Comune di residenza

Il Comune è dipendente dalla Regione selezionata.

Questa relazione è importante perché evita che l'utente possa selezionare, per esempio, Milano dopo avere selezionato una Regione diversa dalla Lombardia.

Il comportamento è gestito dalla funzione:

```javascript
updateMunicipalityOptions()
```

che:

1. legge la Regione;
2. recupera l'elenco dei Comuni appartenenti alla Regione;
3. ricostruisce dinamicamente la select;
4. abilita il campo;
5. aggiorna lo stato mostrato all'utente.

L'elenco dei Comuni è incorporato nel codice attraverso il dataset:

```javascript
ISTAT_MUNICIPALITIES_2026
```

e quindi non viene recuperato da un'API durante l'utilizzo della pagina.

---

## 4.4 Numero di mensilità

L'utente può scegliere:

- 12 mensilità;
- 13 mensilità;
- 14 mensilità.

Il default è 13.

Questo input **non modifica il netto annuale**: modifica esclusivamente la modalità con cui il netto annuale viene trasformato in un netto medio mensile.

La formula utilizzata è:

```text
netto mensile = netto annuale / numero mensilità
```

Questa scelta è intenzionale: il prototipo mostra un **netto medio mensile**, non tenta di simulare l'importo effettivamente presente in ogni singolo cedolino.

---

# 5. Dataset e fonti delle informazioni

Una parte importante del progetto è stata separare, per quanto possibile all'interno di un singolo file HTML, le informazioni normative e territoriali dalla logica di calcolo.

## 5.1 IRPEF

Il codice dichiara di utilizzare per il 2026 i seguenti scaglioni:

| Reddito imponibile | Aliquota |
|---|---:|
| fino a €28.000 | 23% |
| €28.001 – €50.000 | 33% |
| oltre €50.000 | 43% |

Questi valori sono incorporati nella funzione `irpefGross()`.

Il codice applica l'aliquota progressivamente ai diversi intervalli e non applica mai l'aliquota dello scaglione più alto all'intero reddito.

La pagina indica come riferimento normativo il **TUIR, D.P.R. 917/1986**, con le modifiche vigenti per il 2026.

---

## 5.2 Contributi previdenziali

Il prototipo utilizza una contribuzione a carico del lavoratore pari al:

```text
9,19%
```

della RAL.

La scelta viene esplicitamente dichiarata nel codice come stima standard.

Il codice calcola:

```text
contributi = RAL × 9,19%
```

e successivamente:

```text
imponibile fiscale = RAL - contributi
```

Questa è una delle principali semplificazioni del modello.

In un sistema payroll reale l'aliquota contributiva può dipendere da caratteristiche del rapporto, settore, qualifica, contribuzione applicabile e CCNL.

---

## 5.3 Detrazione da lavoro dipendente

La detrazione viene calcolata tramite la funzione:

```javascript
employeeDeduction(income)
```

che implementa tre fasce:

```text
income <= 15.000
income <= 28.000
income <= 50.000
oltre 50.000
```

con le formule presenti direttamente nel codice.

La detrazione viene sottratta dall'IRPEF lorda:

```text
IRPEF netta = max(0, IRPEF lorda - detrazione)
```

L'interfaccia specifica inoltre che la detrazione è riferita all'articolo 13 del TUIR.

---

# 6. Calcolo progressivo delle imposte

Per evitare di implementare separatamente la stessa logica per ogni tipo di addizionale, è stata introdotta una funzione generica:

```javascript
progressiveTax(base, bands)
```

La funzione riceve:

- una base imponibile;
- un array di scaglioni;
- una aliquota associata a ogni scaglione.

Per ogni fascia calcola soltanto la parte di reddito ricadente in quella fascia.

La logica è:

```text
importo dello scaglione =
min(base, limite scaglione) - limite precedente
```

e:

```text
imposta = importo dello scaglione × aliquota
```

Questo rende la funzione riutilizzabile sia per le addizionali regionali sia per quelle comunali.

È una scelta architetturale importante perché evita di duplicare la logica fiscale.

---

# 7. Addizionale regionale

Le aliquote regionali sono memorizzate nel dataset:

```javascript
REGIONAL_RATES_2026
```

Ogni Regione ha una struttura composta da quattro fasce.

Ad esempio, per la Lombardia il codice contiene:

```javascript
Lombardia:
[
  [15000, 0.0123],
  [28000, 0.0158],
  [50000, 0.0172],
  [Infinity, 0.0173]
]
```

La funzione:

```javascript
regionalTax(base, region)
```

passa questi dati alla funzione generica `progressiveTax()`.

L'interfaccia indica come fonte delle aliquote regionali il **Dipartimento delle Finanze / MEF, elenco 2026**.

---

# 8. Addizionale comunale

L'addizionale comunale è modellata attraverso il dataset:

```javascript
COMUNI_2026
```

Per alcuni Comuni sono presenti:

- aliquote;
- scaglioni;
- eventuale soglia di esenzione.

Il codice prevede anche:

```javascript
DEFAULT_COMUNE_2026
```

utilizzato quando il Comune selezionato è presente nell'elenco territoriale ma non è presente nella tabella fiscale specifica.

Questo punto è importante perché il prototipo **non finge di conoscere un'aliquota comunale ufficiale quando non dispone del dato specifico**.

La funzione:

```javascript
municipalTax(base, comune)
```

1. normalizza il nome del Comune;
2. cerca la configurazione fiscale;
3. recupera l'eventuale soglia di esenzione;
4. se il reddito è sotto la soglia, restituisce zero;
5. altrimenti applica il calcolo progressivo.

La UI distingue inoltre tra Comune riconosciuto fiscalmente nel dataset e Comune valido territorialmente ma privo di una configurazione fiscale specifica.

---

# 9. Pipeline completa del calcolo

Il cuore del prodotto è la funzione:

```javascript
calculate()
```

La sequenza logica è la seguente.

## Step 1 — Lettura degli input

Il codice legge:

```text
RAL
mensilità
Regione
Comune
```

Se uno degli input obbligatori manca, il risultato non viene calcolato.

---

## Step 2 — Contributi previdenziali

Si calcola:

```text
contributi = RAL × 9,19%
```

---

## Step 3 — Imponibile IRPEF

Si ottiene:

```text
imponibile = RAL - contributi
```

con protezione:

```text
max(0, RAL - contributi)
```

---

## Step 4 — IRPEF lorda

L'imponibile viene passato alla funzione:

```javascript
irpefGross()
```

che applica progressivamente:

```text
23% fino a 28.000 €
33% da 28.000 € a 50.000 €
43% oltre 50.000 €
```

---

## Step 5 — Detrazione da lavoro dipendente

Il codice calcola:

```text
detrazione = employeeDeduction(imponibile)
```

---

## Step 6 — IRPEF netta

La formula diventa:

```text
IRPEF netta =
max(0, IRPEF lorda - detrazione)
```

---

## Step 7 — Addizionale regionale

Viene calcolata sulla stessa base imponibile:

```text
addizionale regionale =
regionalTax(imponibile, Regione)
```

---

## Step 8 — Addizionale comunale

Analogamente:

```text
addizionale comunale =
municipalTax(imponibile, Comune)
```

---

## Step 9 — Netto annuale

Il netto annuale viene ottenuto sottraendo dalla RAL:

```text
contributi
+ IRPEF netta
+ addizionale regionale
+ addizionale comunale
```

Formula:

```text
Netto annuale =
RAL
- contributi
- IRPEF netta
- addizionale regionale
- addizionale comunale
```

Il codice protegge anche il risultato con `Math.max(0, ...)`.

---

## Step 10 — Netto medio mensile

Infine:

```text
Netto medio mensile =
Netto annuale / numero mensilità
```

Questo valore è quindi una media matematica e non una simulazione dei singoli cedolini.

---

# 10. Rappresentazione del risultato

Il risultato non mostra soltanto il netto finale, ma espone il modello di calcolo.

Vengono visualizzati:

- RAL;
- contributi dipendente;
- IRPEF netta;
- addizionale regionale;
- addizionale comunale;
- detrazione da lavoro dipendente;
- netto annuale.

Ogni valore è inoltre accompagnato, dove utile, dalla sua incidenza percentuale sulla RAL.

Questa scelta è stata fatta per rendere il risultato **spiegabile**, non soltanto numericamente corretto secondo il modello implementato.

---

# 11. Definizione di “totale trattenute”

Il prodotto mostra un indicatore chiamato:

> Totale contributi del dipendente + imposte/addizionali

Il valore è sostanzialmente:

```text
RAL - netto annuale
```

e quindi:

```text
contributi
+ IRPEF netta
+ addizionale regionale
+ addizionale comunale
```

È importante precisare che nel documento e nell'interfaccia non viene interpretato come il “cuneo fiscale” in senso statistico completo.

Non vengono infatti inclusi i contributi a carico del datore di lavoro.

Questa distinzione è esplicitata anche direttamente nella UI.

Per maggiore precisione concettuale, il valore va quindi interpretato come **trattenute a carico del dipendente modellate dal simulatore**, non come costo fiscale/contributivo complessivo del lavoro.

---

# 12. Grafico di composizione

Il risultato contiene anche una barra visuale che rappresenta la composizione della RAL tra:

- netto;
- imposte;
- contributi.

Le larghezze sono calcolate come percentuale della RAL:

```text
netto / RAL
imposte / RAL
contributi / RAL
```

Questa visualizzazione ha principalmente uno scopo comunicativo: permette di comprendere immediatamente quanto della RAL rimane al dipendente e quanto viene assorbito dalle diverse componenti.

---

# 13. Aggiornamento dell'interfaccia

Il calcolo non viene eseguito soltanto quando l'utente clicca “Calcola”.

Sono presenti diversi event listener.

### RAL

Quando cambia la RAL:

```text
input → calculate()
```

### Mensilità

Quando cambia il numero di mensilità:

```text
change → calculate()
```

### Regione

Quando cambia la Regione:

1. il Comune selezionato viene resettato;
2. viene ricostruito l'elenco dei Comuni;
3. vengono aggiornate le informazioni fiscali.

### Comune

Quando cambia il Comune:

1. viene salvato il Comune selezionato;
2. vengono aggiornate le informazioni fiscali;
3. viene eseguito nuovamente il calcolo.

Il pulsante “Calcola” mantiene comunque un ruolo esplicito nell'esperienza utente e porta automaticamente il risultato al centro della viewport.

---

# 14. Gestione dei nomi dei Comuni

I nomi dei Comuni vengono normalizzati attraverso:

```javascript
normalizeName()
```

La funzione:

1. converte in minuscolo;
2. normalizza i caratteri Unicode;
3. rimuove gli accenti;
4. elimina gli spazi iniziali e finali.

Questo permette di confrontare correttamente nomi la cui differenza dipende esclusivamente dalla rappresentazione del testo.

La stessa normalizzazione viene utilizzata per collegare il Comune alla relativa configurazione fiscale.

---

# 15. Mappa regionale

Come funzionalità aggiuntiva è stata introdotta una mappa dell'Italia che confronta il netto annuale tra le 20 Regioni.

Questa funzione non utilizza il Comune.

L'obiettivo è isolare l'effetto dell'addizionale regionale e permettere all'utente di comprendere quanto la Regione di residenza possa incidere sul risultato.

La funzione:

```javascript
regionalAnnualNet()
```

calcola per ogni Regione:

```text
RAL
- contributi
- IRPEF netta
- addizionale regionale
```

escludendo intenzionalmente l'addizionale comunale.

Questa scelta è fondamentale per rendere il confronto tra Regioni omogeneo.

---

# 16. Implementazione tecnica della mappa

Le geometrie della cartina non sono state disegnate manualmente.

Il codice carica dinamicamente:

```text
@svg-maps/italy
```

tramite CDN:

```text
https://cdn.jsdelivr.net/npm/@svg-maps/italy@2.0.0/+esm
```

Il modulo fornisce le geometrie SVG delle Regioni. Il codice crea poi dinamicamente i relativi `<path>` e associa a ciascuna Regione il valore netto calcolato.

È presente anche una gestione del fallimento del caricamento: se la mappa non viene caricata, il calcolatore principale continua comunque a funzionare.

Questo introduce una dipendenza esterna limitata esclusivamente alla visualizzazione della mappa.

---

# 17. Tabella dei CCNL

La pagina contiene anche una sezione informativa dedicata ai principali CCNL.

Sono presenti dati relativi a:

- Terziario, Distribuzione e Servizi — Confcommercio;
- Metalmeccanici Industria — Federmeccanica-Assistal;
- Pubblici Esercizi, Ristorazione Collettiva e Turismo — FIPE;
- Studi Professionali.

I valori mostrati sono minimi contrattuali/tabellari e non rappresentano un “minimo salariale legale” unico.

La pagina precisa infatti che in Italia non esiste un unico salario minimo legale applicabile indistintamente a tutti i lavoratori e distingue i minimi contrattuali dalle altre componenti della retribuzione.

Le fonti dichiarate dal codice sono:

- Confcommercio per il CCNL Terziario;
- tabelle 2026 del CCNL Metalmeccanici Industria;
- Confcommercio Milano/FIPE per Pubblici Esercizi;
- tabelle CCNL Studi Professionali.

---

# 18. Tredicesima e quattordicesima

Il simulatore permette di selezionare 12, 13 o 14 mensilità.

Tuttavia non simula separatamente il cedolino di ogni mensilità.

Questo è importante perché tredicesima e quattordicesima possono avere un trattamento differente rispetto a una mensilità ordinaria.

La pagina lo esplicita chiaramente: il risultato visualizzato è un **netto medio**, e quindi non deve essere interpretato come l'importo effettivo della tredicesima o della quattordicesima.

La semplificazione è quindi:

```text
netto medio = netto annuale / numero di mensilità
```

anziché:

```text
simulazione cedolino gennaio
+ febbraio
+ ...
+ tredicesima
+ eventuale quattordicesima
```

Questa scelta riduce drasticamente la complessità mantenendo il risultato coerente con lo scopo del prototipo.

---

# 19. Assunzioni principali

Il modello è volutamente semplificato.

Le principali assunzioni sono:

## Rapporto di lavoro

Si considera un lavoratore dipendente standard.

Non vengono modellate:

- part-time;
- apprendistato;
- dirigenti;
- categorie particolari;
- rapporti con contribuzioni speciali.

## Contributi

Viene utilizzata un'aliquota standard del 9,19%.

Non viene costruito un motore contributivo dipendente da settore/qualifica/CCNL.

## Fiscalità personale

Non vengono considerate casistiche personali come:

- familiari a carico;
- altre detrazioni;
- altri redditi;
- oneri deducibili;
- bonus;
- welfare;
- premi;
- fringe benefit;
- situazioni fiscali individuali.

## Conguagli

Non viene simulata la dinamica completa dei conguagli fiscali di fine anno.

## Cedolino

Non viene prodotto un cedolino mensile reale.

Il risultato è una proiezione annuale trasformata in media mensile.

---

# 20. Perché queste semplificazioni

La scelta deriva direttamente dalla natura del task.

L'obiettivo non era realizzare un payroll engine completo, ma dimostrare la capacità di:

1. comprendere il dominio;
2. individuare le variabili principali;
3. reperire dati rilevanti;
4. trasformare tali dati in una logica di calcolo;
5. costruire un prototipo funzionante;
6. comunicare chiaramente assunzioni e limiti.

Un motore payroll realmente completo richiederebbe molte più informazioni rispetto alla sola RAL.

Per esempio, la stessa aliquota contributiva può dipendere da caratteristiche del rapporto che non sono disponibili nei quattro input del prototipo.

Per questo è stato preferito un modello semplice ma esplicito rispetto a un modello apparentemente più preciso ma difficile da verificare.

---

# 21. Gestione dei dati mancanti

Un principio importante adottato nella soluzione è quello di **non nascondere l'incertezza**.

Quando un Comune è presente nell'elenco territoriale ma non è presente nel dataset fiscale specifico, il codice utilizza un valore di fallback.

L'interfaccia però non presenta quel dato come se fosse necessariamente l'aliquota ufficiale del Comune.

Questa distinzione è importante dal punto di vista product:

> è preferibile comunicare all'utente che un dato è una stima/fallback piuttosto che fornire un numero apparentemente preciso ma privo di una fonte specifica.

---

# 22. Privacy e architettura client-side

Il prototipo non richiede un server per il calcolo.

RAL, Regione, Comune e mensilità vengono utilizzati direttamente dal JavaScript presente nella pagina.

Non è presente un flusso di invio dei dati a un backend.

Questo comporta due vantaggi per il prototipo:

- semplicità architetturale;
- nessuna necessità di gestire dati personali lato server.

L'unica chiamata esterna presente nel codice è relativa al caricamento della geometria SVG della mappa regionale.

---

# 23. Deployment e pubblicazione online

Per rendere il prototipo immediatamente utilizzabile e verificabile, la pagina è stata pubblicata online tramite **Netlify**.

Il deployment è stato realizzato mantenendo l'architettura client-side descritta nei paragrafi precedenti: non è quindi necessario un backend dedicato per eseguire il calcolo.

### Demo online

https://unique-puffpuff-195472.netlify.app

La pubblicazione online permette di verificare direttamente il comportamento del prodotto senza dover configurare localmente un ambiente di sviluppo.

In particolare, il deployment consente di:

- aprire il simulatore direttamente dal browser;
- inserire una RAL e gli altri parametri;
- verificare il funzionamento del calcolo;
- interagire con la mappa regionale;
- verificare la UI e il comportamento responsive;
- utilizzare il prototipo senza installare dipendenze o avviare un server locale.

La scelta di **Netlify** è coerente con la natura del progetto: trattandosi di una pagina statica con logica JavaScript eseguita lato client, non è necessario introdurre un'infrastruttura backend per il prototipo.

Il deployment rappresenta quindi semplicemente il livello di distribuzione del prototipo:

```text
Codice HTML / CSS / JavaScript
              │
              ▼
           Netlify
              │
              ▼
        Browser dell'utente
              │
              ▼
       Calcolo client-side
```

L'architettura rimane quindi volutamente semplice: Netlify si occupa di rendere disponibile la pagina, mentre il motore di calcolo viene eseguito direttamente nel browser.

---

# 24. Struttura tecnica del file

Il file può essere concettualmente diviso in:

```text
index.html
│
├── <head>
│   ├── metadata
│   └── CSS
│
├── <body>
│   ├── Header
│   ├── Hero
│   ├── Calculator
│   │   ├── RAL
│   │   ├── Regione
│   │   ├── Comune
│   │   └── Mensilità
│   │
│   ├── Result
│   │   ├── Netto mensile
│   │   ├── Breakdown
│   │   ├── Trattenute
│   │   └── Explanations
│   │
│   ├── CCNL information
│   ├── Regional map
│   └── Footer
│
└── <script>
    ├── Dataset fiscali
    ├── Dataset Comuni
    ├── Funzioni di normalizzazione
    ├── Funzioni fiscali
    ├── calculate()
    ├── Event listeners
    └── Regional map
```

La soluzione è quindi volutamente **monolitica** dal punto di vista del file, ma con una separazione logica abbastanza chiara tra dati, calcolo e presentazione.

---

# 25. Principali funzioni JavaScript

## `normalizeName()`

Normalizza i nomi dei Comuni per consentire confronti robusti.

## `updateMunicipalityOptions()`

Popola la select dei Comuni in base alla Regione.

## `getComuneConfig()`

Recupera la configurazione fiscale di un Comune o il fallback.

## `formatRateList()`

Trasforma le aliquote presenti nel dataset in una rappresentazione leggibile per la UI.

## `updateTaxCards()`

Aggiorna le card che mostrano aliquote regionali e comunali.

## `progressiveTax()`

Motore generico per il calcolo progressivo per scaglioni.

## `regionalTax()`

Applica `progressiveTax()` alle aliquote regionali.

## `municipalTax()`

Applica `progressiveTax()` alle aliquote comunali, considerando l'eventuale esenzione.

## `irpefGross()`

Calcola l'IRPEF lorda.

## `employeeDeduction()`

Calcola la detrazione per lavoro dipendente.

## `calculate()`

È la funzione centrale che coordina l'intera pipeline dal dato di input al netto finale.

## `regionalAnnualNet()`

Calcola il netto annuale utilizzato esclusivamente per il confronto tra Regioni.

## `renderRegionalMap()`

Aggiorna mappa e tabella regionale.

---

# 26. Flusso complessivo

Il funzionamento può essere rappresentato così:

```text
                 RAL
                  │
                  ▼
       ┌────────────────────┐
       │ Contributi 9,19%   │
       └─────────┬──────────┘
                 │
                 ▼
       Imponibile fiscale
                 │
       ┌─────────┴─────────┐
       ▼                   ▼
   IRPEF lorda       Addizionali
       │              regionali/comunali
       ▼                   │
   Detrazione              │
       │                   │
       ▼                   │
   IRPEF netta ────────────┘
                 │
                 ▼
        Netto annuale
                 │
                 ▼
        / mensilità
                 │
                 ▼
       Netto medio mensile
```

La formula complessiva è:

```text
Imponibile =
RAL - contributi

IRPEF netta =
IRPEF lorda - detrazione

Netto annuale =
RAL
- contributi
- IRPEF netta
- addizionale regionale
- addizionale comunale

Netto medio mensile =
Netto annuale / mensilità
```

---

# 27. Cosa il prototipo fa bene

Dal punto di vista del test, i principali punti di forza della soluzione sono:

### Trasparenza

Il risultato non è una “black box”: ogni componente viene mostrata separatamente.

### Esplicitazione delle assunzioni

L'utente viene informato che si tratta di una simulazione.

### Dati territoriali

Il Comune è vincolato alla Regione e viene considerata la fiscalità locale.

### Progressive taxation

La logica a scaglioni è implementata in maniera generica e riutilizzabile.

### Funzionamento client-side

Il prototipo può essere aperto direttamente senza infrastruttura backend.

### Resilienza

La mancata disponibilità della mappa non impedisce al calcolatore principale di funzionare.

### Comunicazione

La UI contiene spiegazioni sulle principali componenti fiscali e previdenziali, rendendo il risultato più comprensibile.

### Deployment

Il prototipo è stato pubblicato online tramite Netlify ed è quindi direttamente verificabile tramite browser.

---

# 28. Limiti del prototipo

I limiti non sono bug accidentali: sono principalmente conseguenze delle semplificazioni scelte.

## 28.1 Aliquota contributiva standard

Il 9,19% è una stima e non un motore contributivo completo.

## 28.2 Modello fiscale semplificato

Il modello non rappresenta tutte le casistiche fiscali personali.

## 28.3 Addizionali comunali

Il dataset fiscale comunale incorporato non rappresenta necessariamente l'intera complessità e completezza dei dati MEF.

Il codice stesso prevede un fallback per i Comuni senza configurazione specifica.

## 28.4 Mensilità

Il netto mensile è una media e non una simulazione cedolino-per-cedolino.

## 28.5 CCNL

I CCNL vengono mostrati come riferimento informativo ma non entrano direttamente nel calcolo.

## 28.6 Cuneo fiscale

Il valore mostrato come totale delle trattenute riguarda esclusivamente le componenti a carico del dipendente considerate dal modello.

Non include il costo contributivo del datore di lavoro.

## 28.7 Aggiornamento normativo

Le aliquote sono hardcoded nel file.

In una soluzione di produzione sarebbe preferibile separare i dati normativi dal codice applicativo e aggiornarli attraverso una fonte governata e versionata.

---

# 29. Come evolverei la soluzione verso un prodotto reale

Il prototipo costituisce una base utile, ma un prodotto payroll reale richiederebbe una separazione più netta tra **motore fiscale, dati e interfaccia**.

Una possibile architettura sarebbe:

```text
                 Frontend
                    │
                    ▼
             Calculation API
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
   Tax Engine   Contribution   Location
                Engine          Data
        │
        ▼
   Versioned rules
        │
        ├── IRPEF
        ├── detrazioni
        ├── addizionali
        ├── INPS
        └── altre regole
```

I dati fiscali dovrebbero essere versionati per anno e, quando necessario, per decorrenza.

Il motore dovrebbe inoltre ricevere informazioni più dettagliate:

- tipo di contratto;
- settore;
- qualifica;
- CCNL;
- mensilità;
- eventuale part-time;
- situazione familiare;
- altre detrazioni;
- fringe benefit;
- welfare;
- bonus;
- eventuali agevolazioni;
- altre caratteristiche contributive.

In questo modo si passerebbe da una **stima standardizzata** a un vero motore payroll.

---

# 30. Possibili miglioramenti del prototipo

Se il tempo a disposizione fosse maggiore, le evoluzioni che considererei prioritarie sarebbero:

## Priorità 1 — Separare i dati dalla UI

Spostare:

```text
REGIONAL_RATES_2026
COMUNI_2026
ISTAT_MUNICIPALITIES_2026
```

in file JSON separati.

Questo renderebbe più semplice l'aggiornamento annuale.

## Priorità 2 — Rendere versionate le regole

Invece di avere:

```text
REGIONAL_RATES_2026
```

si potrebbe avere una struttura del tipo:

```text
taxRules[year]
```

per poter gestire 2025, 2026, 2027 ecc.

## Priorità 3 — Test automatici

Il motore fiscale dovrebbe essere separato dalla UI e coperto da test sui casi limite:

- reddito sotto €15k;
- €15k;
- €28k;
- €28.001;
- €50k;
- €50.001;
- RAL molto elevate;
- esenzione comunale;
- Comune senza dato fiscale.

## Priorità 4 — Separare netto annuale e cedolini

Il modello potrebbe produrre:

```text
netto annuale
+
12/13/14 cedolini
```

applicando le regole specifiche alle mensilità aggiuntive.

## Priorità 5 — Motore contributivo

L'aliquota standard dovrebbe essere sostituita da un motore basato sulle caratteristiche effettive del rapporto.

---

# 31. Considerazioni finali

La soluzione è stata progettata come un **prototipo di simulazione**, non come un sostituto di un software payroll.

La scelta principale è stata privilegiare:

- semplicità;
- trasparenza;
- funzionamento immediato;
- esplicitazione delle assunzioni;
- leggibilità delle formule;
- separazione concettuale tra dati e logica;
- possibilità di estendere successivamente il modello.

Il valore del prototipo non è quindi soltanto il numero finale prodotto dal calcolatore, ma il fatto che il percorso che porta dalla RAL al netto sia **comprensibile, ispezionabile e modificabile**.

In particolare, il modello rende esplicito il seguente percorso:

```text
RAL
→ contributi previdenziali
→ imponibile fiscale
→ IRPEF lorda
→ detrazione
→ IRPEF netta
→ addizionale regionale
→ addizionale comunale
→ netto annuale
→ netto medio mensile
```

Questo permette di avere un prototipo semplice ma sufficientemente strutturato per rappresentare il problema e, soprattutto, costituisce una base chiara da cui partire per un eventuale motore payroll più completo.

Il prototipo è stato infine pubblicato online tramite Netlify, trasformando l'implementazione in una demo direttamente utilizzabile e verificabile:

**https://unique-puffpuff-195472.netlify.app**

---

# Appendix — Nota sulle scelte di prodotto

Una scelta volutamente diversa rispetto alla traccia originale è stata quella di **non limitare il prototipo a Milano**.

La traccia proponeva Milano come assunzione possibile per semplificare il problema. Nel prototipo, invece, Regione e Comune sono diventati input.

Questa non è stata una scelta casuale: la residenza fiscale è una variabile che incide realmente sul risultato tramite le addizionali regionali e comunali.

Ho quindi scelto di trasformare un'assunzione della traccia in un input, mantenendo comunque semplificato il resto del modello.

Questa estensione permette di mostrare un aspetto interessante del dominio: le addizionali regionali e comunali incidono sul risultato e quindi la localizzazione è una variabile materialmente rilevante.

Allo stesso tempo, la soluzione mantiene il modello semplice: non tenta di introdurre tutte le variabili payroll possibili, ma estende il caso base lungo una dimensione che ha un impatto diretto sul risultato.

---

# Fonti dichiarate nell'implementazione

Le fonti richiamate direttamente dalla pagina sono:

- **MEF / Dipartimento delle Finanze** — aliquote delle addizionali regionali e comunali;
- **ISTAT** — elenco territoriale dei Comuni;
- **TUIR, D.P.R. 917/1986** — disciplina IRPEF e detrazione da lavoro dipendente;
- **disciplina previdenziale INPS** — riferimento generale per i contributi;
- **CCNL / fonti contrattuali** — per i minimi contrattuali riportati nella sezione informativa;
- **@svg-maps/italy** — geometrie della mappa regionale.

La pagina stessa dichiara queste fonti nel footer e nelle sezioni informative.
