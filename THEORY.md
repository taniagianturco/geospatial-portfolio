# Teoria dell'Interferometria SAR per la Generazione di DEM

## Introduzione

Questo documento spiega i concetti teorici alla base della generazione di un Modello Digitale di Elevazione (DEM) tramite interferometria SAR (InSAR), seguendo il workflow del tutorial ESA/SkyWatch per Sentinel-1.

---

## L'idea di base

Immagina di fotografare la stessa montagna da due punti leggermente diversi. Le due foto sembrano quasi identiche, ma se le sovrapponi vedrai piccole differenze — oggetti vicini sembrano spostarsi più di quelli lontani. Da queste differenze puoi ricavare informazioni sulla distanza e quindi sull'altezza.

L'interferometria SAR funziona esattamente così, ma invece di fotografie usa **onde radar**.

---

## Concetti chiave

### Il segnale SAR
Un segnale SAR contiene due informazioni:
- **Ampiezza**: la forza del ritorno radar (quanto è riflettente la superficie)
- **Fase**: il punto del ciclo dell'onda radar al momento del ritorno al satellite

È la **fase** che contiene l'informazione sull'altezza del terreno.

### La baseline perpendicolare
Le due immagini vengono acquisite dallo stesso satellite in momenti diversi, da posizioni leggermente diverse nello spazio. La distanza tra queste due posizioni, misurata perpendicolarmente alla direzione di volo, si chiama **baseline perpendicolare**.

- Baseline troppo piccola (< 150 m): l'effetto topografico sulla fase è troppo debole
- Baseline ottimale (150–300 m): buona sensibilità alla topografia
- Baseline troppo grande (> 300 m): decorrelazione della fase

Nel dataset di questo progetto (Erzincan, Turchia) la baseline era di circa **190 metri** — un valore favorevole.

### La baseline temporale
Il tempo tra le due acquisizioni si chiama **baseline temporale**. Più è breve, meglio è, perché:
- La vegetazione si muove tra un'acquisizione e l'altra → **decorrelazione temporale**
- Le condizioni atmosferiche cambiano → errori di fase
- Sentinel-1 ha un ciclo di rivisita di 6 o 12 giorni

---

## Il workflow passo per passo

### 1. Apply Orbit File
Prima di tutto si aggiungono informazioni precise sulla posizione esatta del satellite in ogni momento tramite i file orbitali POE (Precise Orbit Ephemerides). Senza questo i calcoli successivi sarebbero imprecisi.

### 2. Back Geocoding (Coregistrazione)
Le due immagini vengono allineate pixel per pixel con precisione sub-pixel. È necessario perché le due immagini, pur essendo quasi identiche, non sono perfettamente sovrapposte a causa della diversa posizione del satellite.

### 3. Enhanced Spectral Diversity (ESD)
Raffina ulteriormente l'allineamento correggendo piccoli errori residui in azimuth tra i burst adiacenti. Necessario quando si selezionano più di un burst in modalità TOPS.

### 4. Interferogram Formation
Questo è il **cuore del processo**. La fase delle due immagini viene sottratta matematicamente:

```
φ_interferogramma = φ_immagine1 - φ_immagine2
```

Il risultato è l'**interferogramma**: un'immagine con pattern colorati ad arcobaleno chiamati **frange**. Ogni ciclo di colori rappresenta una differenza di distanza di mezza lunghezza d'onda radar (~2.8 cm per Sentinel-1 in banda C). Più frange ci sono in un'area, più il terreno cambia quota in quella zona.

Viene anche calcolata la **coerenza** (valori 0-1):
- Alta (> 0.6): zone rocciose, urbane → fase affidabile
- Bassa (< 0.3): vegetazione, acqua → fase inaffidabile

### 5. TOPS Deburst
Rimuove le giunzioni visibili tra i burst adiacenti, producendo un'immagine continua. Necessario per la modalità IW (Interferometric Wide) di Sentinel-1.

### 6. Goldstein Phase Filtering
Riduce il rumore nell'interferogramma usando una trasformata di Fourier (FFT). Rende le frange più nitide e migliora la qualità dell'unwrapping successivo.

### 7. Phase Unwrapping (snaphu)
Il passo più delicato. Il problema è che la fase è **avvolta** (wrapped): i valori vanno solo da -π a +π e poi ricomincia da capo, come un orologio che segna solo da 0 a 12.

```
Fase avvolta:    /\/\/\/\   (valori tra -π e +π)
Fase svolta:    /         (valori continui)
```

snaphu (Statistical-Cost Network-Flow Algorithm for Phase Unwrapping) risolve questa ambiguità integrando le differenze di fase tra pixel vicini, producendo una fase **continua** da cui si può ricavare l'altezza assoluta.

### 8. Phase to Elevation
Converte la fase continua in metri di quota sopra il livello del mare, usando:
- La geometria del satellite (baseline, angolo di incidenza)
- Un DEM di riferimento (SRTM) per calibrare i valori assoluti

### 9. Terrain Correction
Corregge le distorsioni geometriche tipiche del SAR. Il radar guarda il terreno in modo obliquo, causando effetti come:
- **Foreshortening**: i versanti rivolti verso il satellite appaiono compressi
- **Layover**: le cime delle montagne sembrano spostate verso il satellite
- **Shadow**: i versanti opposti al satellite non vengono illuminati

Il risultato viene riproiettato in coordinate geografiche reali (WGS84).

---

## Limiti di questo approccio

| Limite | Causa | Effetto |
|--------|-------|---------|
| Decorrelazione sulla vegetazione | Il fogliame si muove tra le due acquisizioni | DEM impreciso in zone boscose |
| Baseline corte di Sentinel-1 | Satellite progettato per DInSAR, non DEM | Difficile trovare coppie adatte |
| Ritardo atmosferico | Vapore acqueo causa ritardi di fase | Errori sistematici nel DEM |
| Errori di unwrapping | Zone a bassa coerenza | Pattern a griglia nel DEM finale |

> **Nota importante**: a causa delle baseline prevalentemente corte di Sentinel-1 e della rapida decorrelazione della banda C sulla vegetazione, il DEM risultante non è necessariamente più accurato di prodotti liberamente disponibili come SRTM o AW3D30. Il valore principale di questo esercizio è didattico.

---

## Dataset utilizzato

| Parametro | Valore |
|-----------|--------|
| Area | Erzincan, Turchia orientale |
| Immagine 1 (master) | S1A, 08 luglio 2019 |
| Immagine 2 (slave) | S1B, 02 luglio 2019 |
| Baseline temporale | 6 giorni |
| Baseline perpendicolare | ~190 m |
| Sub-swath | IW2 |
| Polarizzazione | VV |

---

## Riferimenti

- Braun, A. (2021): *Retrieval of digital elevation models from Sentinel-1 radar data – open applications, techniques, and limitations*. Open Geosciences, 13(1), 532-569. [doi:10.1515/geo-2020-0246](https://doi.org/10.1515/geo-2020-0246)
- Tutorial originale: *DEM generation with Sentinel-1 — Workflow and challenges*, SkyWatch/ESA, aggiornato giugno 2021
- Documentazione SNAP: [http://step.esa.int](http://step.esa.int)
- snaphu: [https://web.stanford.edu/group/radar/softwareandlinks/sw/snaphu/](https://web.stanford.edu/group/radar/softwareandlinks/sw/snaphu/)
