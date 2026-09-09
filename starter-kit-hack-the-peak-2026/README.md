# Starter kit - Hack The Peak 2026

Tre file, in quest'ordine.

1. `lovable-starter-prompt.md` - i prompt per avere l'applicazione, il database e la
   funzione che manda le schede alla piattaforma. Prima ora.
2. `ingest-function.ts` - il codice della funzione. Si incolla in una edge function di
   Lovable Cloud chiamata `ingest`, senza cambiare i nomi dei campi.
3. `elevenlabs-agent-template.json` - l'agente di partenza, con i cinque tool già
   dichiarati. Prima di importarlo sostituisci `INCOLLA-QUI-LA-TUA-KEY` con la tua key e
   metti l'indirizzo vero della tua edge function al posto del placeholder in `salva_lead`.

## C0: la copia che arriva alla piattaforma

Non esponi niente e non consegni nessun indirizzo. Il giro è questo:

1. l'agente chiama il tool `salva_lead`, che punta alla tua edge function `ingest`;
2. la funzione scrive nel tuo database, quello che la tua console legge;
3. la stessa funzione manda una copia a `POST https://hackthepeak.yellowtest.it/api/ingest` con l'header
   `X-Ingest-Token`;
4. alle 17:00 il giudice legge le copie arrivate, non viene a bussare al tuo progetto.

`https://hackthepeak.yellowtest.it` è l'indirizzo della piattaforma dell'evento. Indirizzo e token li trovi
nella tua area personale, nel riquadro "Dove scrive il tuo agente", e vanno nelle due
variabili d'ambiente del progetto Lovable:

| Variabile | Cosa contiene |
|---|---|
| `HTP_INGEST_URL` | `https://hackthepeak.yellowtest.it/api/ingest` |
| `HTP_INGEST_TOKEN` | Il tuo token personale |

`conversation_id` è la chiave: una seconda copia con lo stesso valore aggiorna la scheda
invece di duplicarla.

## Autoverifica, da fare alle 11:00 e non alle 17:05

Prima il token, che non scrive niente:

```bash
curl -i -H "X-Ingest-Token: IL-TUO-TOKEN" \
  https://hackthepeak.yellowtest.it/api/ingest/echo
```

Deve rispondere `200` con `ok: true` e il tuo codice partecipante. Un `401` vuol dire token
sbagliato o incollato male.

Poi la prova vera: **fai una telefonata al tuo agente e guarda il contatore nella tua area.**
Se sale, il ciclo si chiude. Se resta a zero mentre l'agente parla, qualcosa in mezzo non sta
scrivendo. Le schede arrivate si rileggono da `GET https://hackthepeak.yellowtest.it/api/leads` con lo stesso
header.

## Tre modi di rompersi in silenzio

Nessuno dei tre dà errore, li abbiamo trovati a spese nostre.

- **Placeholder lasciato nell'URL del tool.** ElevenLabs segna il tool come chiamato anche
  se l'host non esiste: la conversazione dice che il lead è stato salvato, il contatore
  resta a zero.
- **Key lasciata al segnaposto.** L'API Casavo risponde `400 invalid_participant_key` e la
  telefonata si ferma lì. Va sostituita in tutti e quattro i tool che parlano con Casavo.
- **Import ripetuti del template.** Ogni import lascia cinque tool nel workspace. Dopo tre
  import ce ne sono quindici con lo stesso nome, e l'agente non usa quello che stai
  modificando.

## Le tre cose che il giudice guarda

- Cosa dice l'agente al telefono, su decine di telefonate identiche per tutti.
- Le schede che la tua applicazione ha mandato alla piattaforma durante quelle telefonate.
- Cosa mostra la console a un agente immobiliare che deve lavorare quel lead.

## Documentazione dell'API Casavo

`https://hackthepeak.yellowtest.it/casavo/docs` - Swagger interattivo, si prova dal browser.
Ogni chiamata vuole l'header `X-Participant-Key` con la tua key, la stessa che metti in `HTP_INGEST_TOKEN`.
