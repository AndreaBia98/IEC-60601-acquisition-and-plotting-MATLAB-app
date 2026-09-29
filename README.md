#  Report metodologico IEC 60601 per pompe peristaltiche

## Caratterizzazione gravimetrica della precisione di una pompa dosatrice secondo IEC 60601-2-24 (portate in ml/min)

| | |
|---|---|
| **Software di acquisizione** | `Scale_Acquisition` (MATLAB App) |
| **Software di analisi** | `IEC_60601_2_24_plotter` (MATLAB App) |
| **Norma di riferimento** | IEC 60601-2-24:1998 – cl. 50.101 e 50.102, eq. (1)–(5), Fig. 103–107, Allegato AA.3 |
| **Unità di portata** | **ml/min** |

---

## 1. Scopo

Valutare la **precisione nel tempo** di una pompa dosatrice misurando la massa erogata con una bilancia e rappresentandola con i grafici previsti dalla IEC 60601-2-24: la curva di avvio (*start-up*) e la **curva a tromba** (*trumpet curve*), che mostra l'errore di portata in funzione della durata della finestra di osservazione.

Il metodo è gravimetrico: la pompa eroga in un contenitore posto su una bilancia, la massa cumulata viene registrata nel tempo e da essa si ricavano portata ed errori percentuali.

---

## 2. Panoramica del processo

```
 Pompa ──► liquido erogato ──► bilancia ──(seriale)──► PC
                                                        │
                                          ┌─────────────▼─────────────┐
                                          │  Scale Acquisition        │
                                          │  lettura seriale (g)      │
                                          │  timestamp PC (h,m,s)     │
                                          │  log CSV → salva .xls     │
                                          └─────────────┬─────────────┘
                                                        │  file .xls: h, m, s, g, t
                                          ┌─────────────▼─────────────┐
                                          │  IEC 60601-2-24 plotter   │
                                          │  tara → sottocampionamento│
                                          │  a S → portata → 4 grafici│
                                          └─────────────┬─────────────┘
                                                        │
                                                 export .xlsx
```

I quattro grafici ricostruiti dal plotter:


<img width="1595" height="958" alt="image" src="https://github.com/user-attachments/assets/a1bfa4c5-4158-4474-b05c-1c36b89c1989" />

| # | Grafico | Asse X | Asse Y | Cosa mostra |
|---|---|---|---|---|
| 1 | **Massa nel tempo** | tempo (s) | massa cumulata (g) | Quanto liquido è stato erogato; la pendenza è la portata media |
| 2 | **Portata nel tempo** | tempo (s) | portata (ml/min) | Portata istantanea ricavata a passo S (5 s di default), con la retta della portata impostata e il limite del transitorio |
| 3 | **Transitorio** (start-up, Fig. 105 della norma) | tempo (s) | portata (ml/min) | Come la pompa raggiunge il regime nei primi istanti |
| 4 | **Curva a tromba** (Fig. 106/107 della norma) | finestra di osservazione P (min) | errore di portata (%) | Precisione della pompa in funzione della scala temporale considerata |

---

## 3. Fase 1 – Acquisizione con *Scale Acquisition*

### 3.1 Cosa fa l'applicazione

- **Connessione seriale** alla bilancia: porta COM scelta dal menu a tendina, 9600 baud, 8 bit, nessuna parità, 1 bit di stop. Il pulsante *Connessione* diventa verde quando il collegamento riesce.
- **Ricezione dati**: ogni riga di testo inviata dalla bilancia (terminatore di riga) viene
  1. mostrata così com'è nel campo *Seriale Bilancia*;
  2. interpretata con un'espressione regolare che estrae il numero (segno, decimali con punto o virgola, eventuale esponente) e lo converte in **grammi**;
  3. visualizzata nel campo *Volume* e sul gauge.
- **Logging**: con *Start Logging* attivo, ogni lettura viene scritta su un CSV temporaneo con il **timestamp dell'orologio del PC** (ora, minuti, secondi con millesimi) e il valore in grammi. Colonne: `h, m, s, g`.
- **Grafici in tempo reale** (aggiornati ogni `dt = 5 s`): massa e portata istantanea, calcolata come `60·Δm/dt` (g/min ≈ ml/min). Servono solo come controllo visivo durante la prova.
- **Salvataggio** (*Salva*): legge il CSV, calcola `t = h·3600 + m·60 + s`, lo azzera sulla prima riga (`t = t − t(1)`) e scrive il file **.xls** con colonne `h, m, s, g, t`. Dopo il salvataggio il CSV temporaneo viene svuotato.

> La frequenza di campionamento dei dati grezzi è quella con cui la bilancia trasmette; l'app non la impone.

### 3.2 Procedura operativa

1. **Allestire il banco**: contenitore di raccolta sulla bilancia, linea della pompa con l'uscita posizionata sotto il pelo libero del liquido raccolto (come indicato per l'apparato di prova, Fig. 104a della norma), liquido di prova di qualità acqua ISO classe III.
2. **Impostare sulla pompa la portata di prova** (Qset, in ml/min).
3. Collegare la bilancia al PC, aprire *Scale Acquisition*, scegliere la porta COM e premere **Connessione**.
4. Verificare che il campo *Seriale Bilancia* e il gauge seguano la lettura della bilancia.
5. Attivare **Start Logging** e avviare la pompa: la norma prevede che il periodo di prova inizi *contemporaneamente* all'avvio dell'apparecchio. Il plotter userà la prima riga come istante zero e come tara.
6. Lasciare erogare per l'intera durata della prova.
7. Disattivare *Start Logging*, premere **Salva** e scegliere il formato **.xls**.

---

## 4. Fase 2 – Analisi con *IEC 60601-2-24 plotter*

### 4.1 Caricamento e parametri

Con **LOAD XLS** si carica il file salvato al passo precedente; il nome del file compare in alto nell'app. Parametri disponibili:

<table class="tg"><thead>
  <tr>
    <th class="tg-0pky">Immagine<br></th>
    <th class="tg-0pky">Parametro</th>
    <th class="tg-0pky">Default</th>
    <th class="tg-0pky">Significato</th>
  </tr></thead>
<tbody>
  <tr>
    <td class="tg-0pky" rowspan="6">
<img width="306" height="606" alt="image" src="https://github.com/user-attachments/assets/05c63a64-f5e3-4fa2-96d8-04cf6e3b4e70" /> <br></td>
    <td class="tg-0pky">Sample interval (s)</td>
    <td class="tg-0pky">5</td>
    <td class="tg-0pky">passo di campionamento <span style="font-weight:bold">**S**</span> usato per l'analisi</td>
  </tr>
  <tr>
    <td class="tg-0pky">Max Time window Duration (min)</td>
    <td class="tg-0pky">31</td>
    <td class="tg-0pky">finestra di osservazione massima <span style="font-weight:bold">**P**</span> della curva a tromba</td>
  </tr>
  <tr>
    <td class="tg-0pky">Transient_time (s)</td>
    <td class="tg-0pky">120</td>
    <td class="tg-0pky">durata del transitorio iniziale da escludere dalla curva a tromba</td>
  </tr>
  <tr>
    <td class="tg-0pky">FlowSet (ml/min)</td>
    <td class="tg-0pky">38</td>
    <td class="tg-0pky">portata impostata **r** (Qset)</td>
  </tr>
  <tr>
    <td class="tg-0pky">Filtred</td>
    <td class="tg-0pky">off</td>
    <td class="tg-0pky">applica un filtro (media mobile) prima della curva a tromba</td>
  </tr>
  <tr>
    <td class="tg-0pky">Automatic Plot on change Variable</td>
    <td class="tg-0pky">off</td>
    <td class="tg-0pky">ridisegna i grafici a ogni cambio di parametro</td>
  </tr>
</tbody></table>


I pulsanti **IEC PLOT** (ridisegna) e **SAVE XLS** (esporta i risultati) completano l'interfaccia.


### 4.2 Pre-elaborazione (comune ai grafici)

1. **Tara e tempo zero**: `g ← g − g(1)`, `t ← t − t(1)`. La massa parte da 0 g e il tempo da 0 s.
2. **Sottocampionamento a S**: si calcola `dt = mediana(diff(t))`, poi `step = round(S/dt)` e si tiene un campione ogni `step` righe.
3. **Portata istantanea** (ml/min): `dg = 60 · gradient(g) / gradient(t)`, con `t` in secondi (il significato del fattore 60 è spiegato al § 5.1).
4. **Transitorio**: si individua il campione più vicino a `Transient_time`; ciò che viene prima è il transitorio, ciò che viene dopo è il regime.
5. **Pendenza a regime**: `a = Δm / Δt` (g/min) tra l'inizio del regime e l'ultimo campione, mostrata come `m = …` sul grafico della massa.

### 4.3 I quattro grafici

#### 4.3.1 Massa nel tempo
Massa cumulata (g) contro tempo (s). I cerchi sono i campioni sottocampionati a S, la linea sono i dati grezzi. In un dosaggio ideale è una retta; la pendenza a regime `m` è la portata media (g/min ≈ ml/min).

#### 4.3.2 Portata nel tempo (sottocampionata a S)
Portata istantanea `dg` (ml/min) calcolata sui campioni a passo S = 5 s. Sono tracciati:
- la portata impostata **Qset** (linea tratteggiata rossa, etichetta *Q_set*);
- il limite del transitorio (linea verticale rossa, etichetta *transient*).

Asse verticale come da norma per il grafico di avvio: da **−0,2·Qset** a **2·Qset**, passo **0,2·Qset**.

#### 4.3.3 Transitorio (start-up)
Ingrandimento della portata istantanea dall'istante 0 fino a `Transient_time`, con la stessa scala verticale e la stessa linea Qset. Mostra come la pompa raggiunge la portata di regime (sovraelongazioni, ritardi di avvio, oscillazioni iniziali).

#### 4.3.4 Curva a tromba (trumpet)
Si usano solo i campioni **dopo** il transitorio. Per ogni durata di finestra P (da 1 min fino a `Max Time window`, a passo 1 min) si calcolano:
- **Ep(max)**: massimo errore percentuale medio su tutte le finestre di durata P;
- **Ep(min)**: minimo errore percentuale medio su tutte le finestre di durata P.

Sono tracciate le due curve (asse X = P in min, asse Y = errore %), la linea dello zero e una linea orizzontale etichettata con il valore medio delle due curve. Scala come da norma: **±15 %**, passo **5 %**, asse X da 0 a 31 min.

**Come si legge**: per P piccolo la finestra vede le fluttuazioni istantanee (pulsazione dei rulli della pompa, rumore della bilancia) e la tromba è larga; al crescere di P le fluttuazioni si mediano e le due curve si chiudono verso l'errore medio della pompa. Una tromba stretta e vicina allo zero indica una pompa precisa sia nel breve sia nel lungo periodo; una tromba che si chiude su un valore diverso da zero indica un errore sistematico (tarature, portata media diversa dall'impostata).

---

## 5. Il calcolo IEC in ml/min

### 5.1 Algoritmo della curva a tromba, passo per passo

Dati: masse campionate `W0 … Wn` a intervallo S (min), portata impostata r (ml/min), durata dell'analisi Tx (min), finestra P (min).

**1. Portata per ogni intervallo** (eq. 1 in ml/min)
```
Qi = (Wi − Wi-1) / (S · d)              i = 1 … n
```

**2. Errore percentuale per ogni intervallo**
```
ei = 100 · (Qi − r) / r
```

**3. Numero di finestre di durata P** che stanno nell'analisi
```
m = (Tx − P) / S + 1
```

**4. Errore medio in ciascuna finestra j** (eq. 2 e 3 senza MAX/MIN). La finestra j contiene P/S campioni consecutivi:
```
Ep(j) = (S / P) · Σ ei          con i = j … j + P/S − 1
```
Poiché la somma ha esattamente P/S termini, `Ep(j)` è la **media** degli errori nella finestra.

**5. Massimo e minimo su tutte le finestre**
```
Ep(max) = max over j=1..m di Ep(j)
Ep(min) = min over j=1..m di Ep(j)
```

**6. Ripetere per ogni P** (1, 2, 3, … 31 min). I punti (P, Ep(max)) e (P, Ep(min)) formano le due curve della tromba.

**7. Errore medio complessivo** (eq. 4), su tutto il periodo di analisi:
```
Q = (W_fine − W_inizio) / (T · d)       [ml/min]
A = 100 · (Q − r) / r                    [%]
```

**Verifica utile**: sommando le portate su una finestra la somma "telescopa" e resta solo la differenza di massa agli estremi. Quindi
```
Ep(j) = 100 · ( ΔW_finestra / (P · d) − r ) / r
```
cioè l'errore di ogni finestra si può calcolare direttamente dalle masse, senza passare dalle Qi.

**Condizione di validità**: P/S deve essere un numero intero, quindi S deve dividere esattamente 60 s (es. 1, 2, 3, 5, 6, 10, 12, 15, 20, 30 s).

---

## 6. Dati prodotti

Con **SAVE XLS** il plotter genera un file `.xlsx` (nome proposto: `<file>_IEC60601`) con un foglio per ciascun risultato:

| Foglio | Contenuto |
|---|---|
| `Flowrate` | tempo (s) e portata (ml/min) |
| `Mass` | tempo (s) e massa (g) |
| `P` | durate delle finestre di osservazione (min) |
| `Ep_max` / `Ep_min` | errore % massimo / minimo per ogni P |
| `Qi` | portata istantanea post-transitorio |
| `Window_size_minutes` | intervallo di campionamento S (min) |
| `Time_minutes` | durata totale dell'analisi Tx (min) |

---

## 9. Riferimenti

- IEC 60601-2-24:1998, *Medical electrical equipment – Part 2-24: Particular requirements for the safety of infusion pumps and controllers*
  - cl. 50.101: accuratezza dichiarata e definizione dei termini (S, T, T0, T1, T2, P, Ep, densità d)
  - cl. 50.102: procedura di prova, scale dei grafici
  - eq. (1): portata istantanea · eq. (2)–(3): Ep(max), Ep(min) 
  - Fig. 103: periodi di analisi · Fig. 104a/b: apparato di prova · Fig. 105–107: grafico di avvio e curve a tromba
  - Allegato AA.3: motivazione dell'algoritmo trumpet
