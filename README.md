# Cesports

Gestionale full-stack per società di calcio e rugby. L'interfaccia è realizzata con React e TypeScript, l'API con Java 21 e Spring Boot, e i dati sono conservati in PostgreSQL.

## Avvio locale

Requisiti: Docker Desktop con Docker Compose.

1. Copia `.env.example` in `.env` e imposta password e chiave JWT sicure. La password dell'admin deve avere almeno 12 caratteri; la chiave JWT almeno 32 byte.
2. Avvia i servizi dalla cartella del progetto:

   ```powershell
   docker compose up --build
   ```

3. Apri <http://localhost:5173> e accedi con `CESPORTS_ADMIN_USERNAME` e `CESPORTS_ADMIN_PASSWORD` impostati in `.env`.

L'API è disponibile su `http://localhost:8080`. PostgreSQL non è esposto direttamente alla rete host. Il volume `cesports-data` conserva i dati tra riavvii. `docker compose down -v` rimuove anche il database: eseguilo solo se vuoi cancellare tutti i dati.

Se `.env` non è stato creato, Compose usa credenziali locali di sviluppo (`admin` / `change-me-now-please`) e una chiave JWT di sviluppo. Non usare questi valori in un ambiente reale.

## Funzioni e ruoli

- Dashboard per la squadra attiva, rose, allenamenti, presenze e partite.
- Admin: crea account e gestisce tutti i dati del club.
- Dirigente: gestisce squadre e dati sportivi.
- Allenatore: gestisce giocatori, allenamenti, presenze e partite.
- Gli account creati dall'admin appartengono allo stesso club e condividono i dati.
- Password salvate con BCrypt; API protetta con token JWT. I token durano 8 ore.
- Report chip: importa CSV UTF-8, associa identificativi e colonne numeriche e consulta le medie per allenamento in dashboard.

## API principale

- `POST /api/auth/login`, `GET /api/auth/me`
- `GET|POST /api/teams`, `DELETE /api/teams/{teamId}`
- `GET|POST /api/teams/{teamId}/players`
- `GET|POST /api/teams/{teamId}/trainings`
- `PUT /api/teams/{teamId}/trainings/{trainingId}/attendance`
- `GET|POST /api/teams/{teamId}/chip-reports`
- `PUT /api/teams/{teamId}/players/{playerId}/chip-code`
- `GET|POST /api/teams/{teamId}/matches`
- `GET|POST /api/users` (solo admin)

Le richieste protette usano `Authorization: Bearer <token>`. Il backend applica i permessi e limita ogni account ai dati del proprio club.

## Importazione report chip

Nella sezione **Report chip** carica un CSV UTF-8 (massimo 2 MB, 10.000 righe e 100 colonne), scegli l'allenamento e la colonna che identifica il giocatore/chip, poi verifica l'anteprima e le associazioni. Il sistema preseleziona solo corrispondenze univoche per codice chip, nome o numero di maglia; le altre richiedono una scelta esplicita. Le nuove associazioni vengono salvate nella rosa e riutilizzate per i report successivi.

Le colonne numeriche sono proposte come metriche: puoi selezionare quelle da importare. Ogni riga viene associata al giocatore e all'allenamento scelto; le presenze manuali non vengono modificate, perché l'assenza di una rilevazione non dimostra che un atleta fosse assente. Se il CSV non contiene misure numeriche, la dashboard mostra il numero di giocatori e righe importati senza inventare metriche. Quando ci sono misure, la dashboard mostra le medie delle righe con i nomi delle colonne e il numero di rilevazioni. Unità e significato dipendono dal report esportato: controllali prima di interpretare i valori. Al momento sono supportati CSV, non Excel/PDF.

## Dati della demo precedente

La demo MVP 0.1 memorizzava i dati nel `localStorage` del browser. Come richiesto, il nuovo database parte vuoto e non importa automaticamente quei dati. La demo legacy non fa più parte dell'applicazione.

## Sviluppo senza Docker

- Frontend: Node.js 20 o successivo, `npm ci`, `npm run dev`.
- Backend: Java 21 e Maven, dalla cartella `backend` esegui `mvn spring-boot:run`.
- Avvia PostgreSQL e imposta `SPRING_DATASOURCE_URL`, `SPRING_DATASOURCE_USERNAME`, `SPRING_DATASOURCE_PASSWORD`, `CESPORTS_ADMIN_USERNAME`, `CESPORTS_ADMIN_PASSWORD` e `CESPORTS_JWT_SECRET`. Per il frontend, imposta `VITE_API_URL` se l'API non è su `http://localhost:8080`.
