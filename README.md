# Relazione Progetto Elasticsearch - Sistema di Indicizzazione e Ricerca Film

## URL del Progetto
**GitHub Repository**: https://github.com/AlessandroSchmitt/elasticSearch
## Descrizione del Progetto
Il progetto implementa un sistema di indicizzazione e ricerca per un dataset di film utilizzando Elasticsearch. Il sistema permette di indicizzare file di testo contenenti le trame dei film e di effettuare ricerche avanzate sui contenuti.

## Analyzer Utilizzati

### Scelta degli Analyzer
Per questo progetto è stato scelto l'analyzer **`english`** sia per il campo `title` che per il campo `content`. Questa scelta è motivata da:

```json
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text", 
        "analyzer": "english", 
        "search_analyzer": "english"
      },
      "content": {
        "type": "text", 
        "analyzer": "english", 
        "search_analyzer": "english"
      }
    }
  }
}
```

### Motivazioni della Scelta

1. **Lingua del Dataset**: Il dataset contiene film prevalentemente in lingua inglese, rendendo l'analyzer `english` la scelta più appropriata.

2. **Stemming Automatico**: L'analyzer `english` include automaticamente:
   - **Stop words removal**: Rimuove parole comuni come "the", "and", "or", "in" che non aggiungono valore semantico
   - **Stemming**: Riduce le parole alla loro radice (es. "running" → "run", "movies" → "movie")
   - **Lowercase normalization**: Converte tutto in minuscolo per migliorare il matching

3. **Semplicità e Efficacia**: L'analyzer predefinito `english` fornisce un buon bilanciamento tra semplicità di configurazione e qualità dei risultati di ricerca.

## Indicizzazione

### Numero di File Indicizzati
- **Dataset originale**: Film estratti da `Movies.csv`
- **File indicizzati**: 2976 file `.txt` 

### Processo di Indicizzazione

1. **Preparazione Dataset**:
   - Lettura del CSV con dataset completo di film
   - Sottocampionamento di 2976 record per testing
   - Selezione colonne rilevanti: `Title` e `Plot`

2. **Sanitizzazione Nomi File**:
   ```python
   def safe_filename(name):
       """Rimuove caratteri non validi da un nome di file."""
       name = re.sub(r'[\\/*?:"<>|]', "", name)
       name = re.sub(r"\s+", " ", name).strip()
       return name
   ```

3. **Creazione File**:
   - Ogni film diventa un file `.txt`
   - Nome file basato sul titolo sanitizzato
   - Contenuto del file rappresenta la trama

4. **Indicizzazione Bulk**:
   - Utilizzo di `helpers.bulk()` per indicizzazione di massa
   - Timeout impostato a 120 secondi per gestire grandi volumi
   - Struttura documento:
     ```json
     {
       "_index": "moviesindex",
       "_source": {
         "title": "nome_film_senza_estensione",
         "content": "trama_del_film"
       }
     }
     ```

### Tempi di Indicizzazione
**Nota**: I tempi esatti dipendono dalla configurazione hardware e di rete. Il codice è predisposto per misurare automaticamente i tempi:

```python
start = time.perf_counter()
success, _ = helpers.bulk(es.options(request_timeout=120), actions)
elapsed = time.perf_counter() - start
print(f"Tempo totale: {elapsed:.2f} secondi")
```

Stimati per 2976 documenti:
- **Tempo previsto**: 10-30 secondi (dipendente dalla configurazione)
- **Throughput**: ~100-300 documenti/secondo

## Query di Test

### Esempi di Query di Test

1. **Ricerca per Titolo**:
   - `title: Clockwork` → Trova "A Clockwork Orange"
   - `title: "Billy Elliot"` → Ricerca esatta del titolo

2. **Ricerca per Contenuto**:
   - `content: robot future` → Film sci-fi con robot
   - `content: "love story"` → Film con storie d'amore

3. **Ricerche Complesse**:
   - `content: detective murder mystery` → Film gialli/polizieschi
   - `content: "New York"` → Film ambientati a New York


## Considerazioni Tecniche

### Gestione dei Caratteri Speciali
La funzione `safe_filename()` gestisce caratteri problematici nei nomi dei file, mantenendo la compatibilità del filesystem.

### Robustezza del Sistema
- **Timeout configurabile** per operazioni bulk
- **Verifica esistenza indice** prima della creazione
- **Gestione errori** di connessione a Elasticsearch
- **Encoding UTF-8** per supporto caratteri internazionali

### Scalabilità
Il sistema è progettato per gestire dataset di dimensioni maggiori modificando semplicemente il parametro `n` nel campionamento del dataset.

## Conclusioni
Il sistema implementato fornisce una soluzione efficace per la ricerca testuale su dataset di film, con un buon bilanciamento tra semplicità di implementazione e qualità dei risultati di ricerca. L'uso dell'analyzer `english` si dimostra appropriato per il dominio applicativo, mentre l'architettura modulare permette facilmente estensioni future.
