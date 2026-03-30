# EvenUp — Splitwise‑inspired expense splitting (Semesterprojekt)

**EvenUp** ist eine Web‑Applikation, mit der Gruppen (z. B. „WG“, „Urlaub“, „Verein“) gemeinsame Ausgaben erfassen, fair aufteilen und **Salden** berechnen können: **Wer schuldet wem wie viel?** Zusätzlich gibt es ein „**Schulden vereinfachen**“ : Es verändert niemandes Gesamtsaldo, reduziert aber die Anzahl notwendiger Ausgleichszahlungen.

---

## Projektbeschreibung

Wenn mehrere Personen zusammen wohnen oder reisen, entstehen viele Ausgaben (Einkauf, Miete, Tickets, Restaurants). Oft zahlt eine Person vor, die Aufteilung ist unklar und am Ende wird „Pi mal Daumen“ abgerechnet. EvenUp löst genau dieses Problem:

- Ausgaben werden in **Gruppen** organisiert
- Jede Ausgabe kann **verschieden** gesplittet werden (gleich/ungleich/Prozent/Anteile)
- Die App zeigt jederzeit die **Netto‑Salden** pro Person
- Ein „**Ausgleichen**“ wird als Zahlung erfasst, sodass die Historie nachvollziehbar bleibt
- Payment‑Integrationen via PayPal

**Zielgruppe:** WG‑Bewohner:innen, Freundesgruppen auf Reisen, Paare, Teams/Vereine

---

## Kernfeatures

### Kontakte hinzufügen

### Gruppen & Mitglieder

- Gruppen erstellen (z. B. „Ski Trip 2026“, „WG Sonnenstraße“)
- Mitglieder hinzufügen (Invite link oder direktes adden von Kontakten)
- Jedes Mitglied hat die selben Rechte

### Ausgaben erfassen

- Neue Ausgabe: Titel, Betrag, Datum (Automatisch heute), Kategorie(Automatisch erkennen, manuelles ändern möglich), Notiz
- Zahler auswählen
- Split‑Varianten:
  - **Gleichmäßig**
  - **Ungleich** nach Beträgen
  - **Prozent**
  - **Anteile/Shares**

### Salden, Ausgleich, Historie

- Gruppensaldo‑Übersicht (Netto je Person + „wer schuldet wem“)
- Aktivitätsfeed (Ausgaben + Ausgleichszahlungen)
- „Ausgleichen“: Zahlung erfassen (Bar/Überweisung/PayPal etc.)

### Schulden vereinfachen (Simplify debts)

- Option pro Gruppe: **Zahlungsvorschläge mit minimaler Transaktionsanzahl**, ohne die Netto‑Salden zu verändern

---

## Nice‑to‑have / Stretch goals

- **Wiederkehrende Ausgaben** (Miete/Abos)
- **Suche & Filter** (Mitglied/Kategorie/Zeitraum)
- **Ausgabenanhänge** (Belegfoto)
- **KI Erkennung von Belegen**
- **Charts/Statistiken**
- **Währungen**
- **Offline‑freundlich** (Caching/Optimistic UI)

---

## Tech Stack (geplant)

### Frontend

- **React + TypeScript**
- **Vite** (Tooling)
- **React Router**
- **Tailwind CSS** (modernes Utility‑First Styling, schnelles responsive UI)
- Data Fetching: `fetch` oder **Axios**
- Shared State: **React Context** (mind. ein Feature: Auth/Session, Theme oder Active Group)

### Backend

- **Node.js** + **Express** *oder* **Fastify**
- Input‑Validation: z. B. **Zod**
- Auth: **JWT**

### Datenbank

- **PostgreSQL**
- ORM: **Prisma**

### Testing

- Frontend: **Vitest + React Testing Library**
- Backend: **Vitest + supertest** (API‑Tests)

### Deployment (M5)

- Frontend: Vercel/Netlify
- Backend: Render/Fly.io
- DB: Managed Postgres (Render/Fly/Supabase)

---

## Setup

> Sobald das Projekt initialisiert ist, werden diese Kommandos hier exakt dokumentiert.

### Frontend (Vite)

```bash
cd apps/web
npm install
npm run dev
```

### Backend (API)

```bash
cd apps/api
npm install
npm run dev
```

### Datenbank (Postgres + Prisma)

```bash
cd apps/api
npx prisma migrate dev
npx prisma studio
```

---

## Geplante Projektstruktur

```
/
  apps/
    web/                # React + TS (Vite)
    api/                # Node API (Express/Fastify)
  packages/
    shared/             # shared types/schemas (optional)
  docs/                 # Skizzen, Diagramme, Milestone-Mapping
  docker-compose.yml
  README.md
```

### Frontend (apps/web)

```
src/
  app/                  # Router, Provider, Layout
  pages/                # Routen: Groups, GroupDetail, Settings, Login/Register
  components/           # UI-Bausteine (Buttons, Modals, Forms, Lists)
  features/
    auth/
    groups/
    expenses/
    settlements/
  lib/
    apiClient.ts        # HTTP client + Fehlerbehandlung
    money.ts            # Formatierung, Roundings, cents
    split.ts            # equal/percent/shares/exact
    simplify.ts         # debt simplification
```

### Backend (apps/api)

```
src/
  server.ts
  routes/
    auth.ts
    groups.ts
    expenses.ts
    settlements.ts
  middleware/
    requireAuth.ts
  services/
    balanceService.ts   # Salden & Vorschläge
  db/
    prisma.ts
  tests/
```

---

## Datenmodell (high‑level)

Die Daten sind stark relational — daher Postgres:

- **User**: id, name, email, passwordHash, createdAt
- **Group**: id, name, ownerId, createdAt, simplifyDebtsEnabled
- **GroupMember**: groupId, userId, role
- **Contacts**: userId1, userId2
- **Expense**: id, groupId, createdBy, description, amountCents, currency, paidByUserId, date, category
- **ExpenseSplit**: expenseId, userId, owedCents *(oder percent/shares + abgeleitete owedCents)*
- **Settlement**: id, groupId, fromUserId, toUserId, amountCents, date, note

**Wichtig:** Beträge als **Integer‑Cents** speichern (kein Float), sonst passieren Rundungsfehler.

---

## API‑Design

### Auth

- `POST /auth/register`
- `POST /auth/login`
- `POST /auth/logout` *(falls cookie-based JWT / session-like)*
- `GET /me`

### Gruppen

- `GET /groups`
- `POST /groups`
- `GET /groups/:groupId`
- `POST /groups/:groupId/members`

### Ausgaben

- `GET /groups/:groupId/expenses`
- `POST /groups/:groupId/expenses`
- `DELETE /groups/:groupId/expenses/:expenseId`

### Salden & Ausgleich

- `GET /groups/:groupId/balances`
- `GET /groups/:groupId/settle-up?simplify=true|false`
- `POST /groups/:groupId/settlements`

---

## Roadmap

### M1 — Projektstart & Fundament (statischer Prototyp)

**Ziel:** Schöner HTML/CSS‑Prototyp ohne JavaScript.

- Semantische Seitenstruktur: `header`, `nav`, `main`, `section`, `article`, `footer`
- Prototyp‑Seiten (statisch):
  - Gruppenübersicht
  - Gruppendetail (Ausgabenliste + Salden)
  - „Ausgabe hinzufügen“ Formular
- Responsives Layout (Flex/Grid) + konsistente Typo/Farben + mind. eine Media Query
- URL‑Bewusstsein: saubere Pfade dokumentiert (z. B. `/groups`, `/groups/:id`, `/groups/:id/new-expense`)

### M2 — React‑Umbau & Interaktion

**Ziel:** M1‑UI als React/TS‑App mit echter Interaktion.

- Vite + React + TypeScript (TS aktiv in zentralen Dateien)
- Sinnvolle Komponenten + Props
- `useState` für lokale Interaktion:
  - Ausgabeformular → neue Ausgabe erscheint in Liste
  - Split‑Modus umschaltbar (equal/exact/percent/shares)
- `useEffect` sinnvoll (z. B. demo load, localStorage, initial fetch mock)

### M3 — Daten, Routing & REST

**Ziel:** Navigierbare SPA mit Fetching + Shared State.

- React Router: mind. 2–3 Routen (z. B. `/groups`, `/groups/:groupId`, `/settings`)
- Datenfetching via HTTP:
  - **GET**: Ausgaben/Gruppen anzeigen
  - **POST/PUT/DELETE**: neue Ausgabe oder Settlement erstellen
- Sichtbare Loading/Fehlerzustände
- Shared State via Context (mind. ein Feature):
  - Auth/Session, Theme, oder aktive Gruppe

### M4 — Qualität & Backend

**Ziel:** Eigener Node‑Server + DB + Auth + Tests.

- Express/Fastify API
- Persistenz via Postgres (Prisma migrations)
- JWT‑Auth: Registrierung/Login, geschützte Endpunkte
- 3–5 Tests (mindestens):
  - Split‑Berechnung (Rundung)
  - Balance‑Berechnung aus Expenses + Settlements
  - Simplify‑Debts: weniger Transfers, gleiche Netto‑Salden
  - API Auth‑Guard / Permissions

### M5 — Betrieb & Feinschliff

**Ziel:** Öffentlich erreichbar oder reproduzierbar startbar + mind. 1 Performance/HTTP‑Aspekt.

- Deployment (Render/Fly/Vercel) **oder** Docker Compose
- Performance/HTTP‑Maßnahme dokumentieren:
  - z. B. Compression, Cache‑Header für Assets, Lazy Loading, ETags
- README final: Setup, Testuser, Einschränkungen (keine Secrets im Repo)
- PDF‑Ausarbeitung (~20 Seiten) + Screenshots

---

## Milestone‑Mapping: Kriterium → wo im Code

> Dieser Abschnitt wird pro Meilenstein konkretisiert, sobald die Dateien existieren. Vorlage:

### M1

- **HTML semantisch**: `apps/web-static/*.html` (header/nav/main/section/article/footer)
- **Formular mit label+name**: `apps/web-static/add-expense.html`
- **CSS responsive**: `apps/web-static/styles.css` (Flex/Grid + Media Query)
- **URL‑Struktur**: README Abschnitt „URL‑Design“

### M2

- **TS aktiv genutzt**: `apps/web/src/...`
- **Komponenten/Props**: `apps/web/src/components/...`
- **State/Hooks**: `apps/web/src/features/expenses/...`

### M3

- **Router**: `apps/web/src/app/router.tsx`
- **Fetch GET/POST**: `apps/web/src/lib/apiClient.ts`
- **Loading/Error**: `apps/web/src/components/...`
- **Context**: `apps/web/src/app/providers/...`

### M4

- **Backend Endpunkte**: `apps/api/src/routes/...`
- **DB Schema/Migrations**: `apps/api/prisma/schema.prisma`
- **Auth**: `apps/api/src/routes/auth.ts`, `apps/api/src/middleware/requireAuth.ts`
- **Tests**: `apps/api/src/tests/...` und/oder `apps/web/src/**/__tests__/`**

### M5

- **Deployment/Docker**: `docker-compose.yml` + README „Deployment“
- **Performance‑Maßnahme**: (konkret dokumentieren)

---

## Bekannte Einschränkungen (Platzhalter)

- (z. B. Nur Euro als Währung)

---

