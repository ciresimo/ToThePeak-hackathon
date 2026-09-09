# Come partire su Lovable senza perdere la prima ora

La prima ora decide la giornata. Se alle 11:00 le tue schede non arrivano alla piattaforma,
finisci senza i venti punti del ciclo chiuso, e non si recuperano.

## Prompt uno: la base

Incolla questo come primo messaggio in un progetto Lovable nuovo.

> Crea un'applicazione per la console di un agente immobiliare che riceve lead da un
> front desk vocale. Serve Lovable Cloud con database.
>
> Tabella `leads` con questi campi: conversation_id, name, phone, address, city, sqm (int),
> floor (int), elevator (bool), condition, energy_class, year_built (int), ownership,
> readiness_class, sale_reason, sale_next_step, already_found_new_home (bool),
> other_agency_mandate (bool), tried_selling_alone (bool), timeline_declared_months (int),
> timeline_real_months (int nullable), blocker, evidence (jsonb), valuation_low (int),
> valuation_high (int), comparables (jsonb), agent_id, slot_id, appointment_status,
> appointment_reason, outcome, consent (bool), opt_out (bool), opt_out_history (bool),
> consent_reconfirmed (bool), created_at.
>
> Poi una pagina sola con due colonne: a sinistra la coda dei lead con nome, città,
> classe di maturità e stato dell'appuntamento; a destra la scheda del lead selezionato
> con tutti i dati, la forchetta di valutazione, il progetto di vendita, le evidenze e
> la trascrizione. Nessun login.

## Prompt due: la funzione che riceve i lead e manda la copia

> Crea una edge function chiamata `ingest` che accetta POST con il JSON di un lead.
> Usa esattamente questo codice.

Poi incolla `ingest-function.ts` di questa cartella.

La funzione fa due cose, in quest'ordine: scrive la scheda nel tuo database, che è quello che
la tua console legge, e ne manda una copia alla piattaforma dell'evento, che è quello che
legge il giudice. La copia è la parte che non si può saltare: alle 17:00 il giudice non viene
a bussare al tuo progetto, legge quello che gli è arrivato.

Imposta due variabili d'ambiente nel progetto Lovable, con questi nomi esatti:

| Variabile | Cosa contiene |
|---|---|
| `HTP_INGEST_URL` | L'indirizzo dello store, `https://hackthepeak.yellowtest.it/api/ingest` |
| `HTP_INGEST_TOKEN` | Il tuo token personale |

Le copi tutte e due dalla tua area sulla piattaforma, dal riquadro "Dove scrive il tuo
agente". Non serve dichiarare da nessun'altra parte chi sei: è il token che ti identifica.
È un segreto, chi ce l'ha scrive nella tua area.

`conversation_id` è la chiave. Una seconda copia con lo stesso `conversation_id` aggiorna la
scheda invece di crearne un'altra: serve davvero, uno degli scenari è una telefonata che cade
e riprende, e una seconda scheda costa punti.

## Prompt tre: il tool dell'agente

Nel template ElevenLabs il tool `salva_lead` ha un placeholder al posto dell'indirizzo. Mettici
l'indirizzo vero della tua edge function, che è
`https://<ref>.supabase.co/functions/v1/ingest`, dove `<ref>` è il codice del tuo progetto
Supabase: lo leggi dall'indirizzo che Lovable ti mostra per la funzione.

Il token dello store **non** va nel tool: sta nelle variabili d'ambiente della funzione. Il
tool parla solo con la tua funzione.

## Verifica prima di andare avanti

Prima il token, con una chiamata che non scrive niente:

```bash
curl -i -H "X-Ingest-Token: IL-TUO-TOKEN" \
  https://hackthepeak.yellowtest.it/api/ingest/echo
```

Deve rispondere `200` con `ok: true` e il tuo codice partecipante. Un `401` vuol dire token
sbagliato o incollato male.

Poi la prova vera, che è una sola: **fai una telefonata al tuo agente e guarda il contatore
nella tua area.** Se sale, il ciclo si chiude e i venti punti sono al sicuro. Se resta a zero
mentre l'agente parla, qualcosa in mezzo non sta scrivendo, e lo scopri adesso invece che alle
17:05. Le schede arrivate si rileggono da `GET https://hackthepeak.yellowtest.it/api/leads` con lo stesso header.

## Cosa fa perdere tempo, per esperienza

- **Lasciare il placeholder nell'URL di `salva_lead`.** ElevenLabs segna il tool come chiamato
  anche quando l'host non esiste: la conversazione dice che il lead è stato salvato, il
  contatore resta a zero e non hai motivo di guardarlo.
- **Lasciare il segnaposto al posto della key.** L'API Casavo risponde `400
  invalid_participant_key` e la telefonata si ferma lì: la key vera comincia per `htp_ing_`
  e sta nella tua area personale. Va messa in tutti e quattro i tool che parlano con Casavo.
- **Reimportare il template senza pulire.** Ogni import lascia cinque tool nel workspace: dopo
  tre import ce ne sono quindici con lo stesso nome, e l'agente non usa quello che stai
  modificando.
- Rinominare i campi. Il giudice cerca i nomi di questo documento, non i tuoi sinonimi.
- Salvare `sqm` come stringa. Deve essere un numero.
- Scrivere `timeline_real_months` uguale al dichiarato quando la telefonata lo smentisce.
  Su quei casi si perde il blocco più pesante.
- Costruire l'interfaccia prima del ciclo dati. L'interfaccia la fa Lovable in dieci minuti
  alle 15:00, il ciclo dati no.
