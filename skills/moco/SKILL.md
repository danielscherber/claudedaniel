---
name: moco
description: >
  MOCO Agentursoftware — API v1, Webhooks, n8n-Integration und MCP Server.
  Use this skill whenever the user asks about MOCO: API-Aufrufe, Zeiterfassung,
  Projekte, Rechnungen, Webhooks, n8n-Workflows mit MOCO, oder den MOCO MCP Server.
  Trigger on: "MOCO", "mocoapp", "MOCO API", "MOCO Webhook", "MOCO n8n", "MOCO MCP".
---

# MOCO Skill

Dieses Skill deckt die MOCO Agentursoftware ab, mit Fokus auf:

- **REST API v1** — Authentifizierung, alle Endpunkte, Pagination, Rate Limits
- **Webhooks** — Event-basierte Benachrichtigungen
- **n8n-Integration** — HTTP Request Nodes mit MOCO API
- **MCP Server** — Claude-Integration via MOCO MCP

Für tiefere Details lade die passende Referenzdatei:
- `references/api.md` — Vollständige API-Referenz aller Endpunkte
- `references/webhooks.md` — Webhook-Setup, Events, Signaturprüfung
- `references/n8n.md` — n8n-Workflows und Patterns mit MOCO
- `references/features.md` — MOCO-Funktionsübersicht (Deutsch)

---

## Authoritative Sources — Always Search First

| Source | Was zu finden |
|---|---|
| `https://everii-group.github.io/mocoapp-api-docs/` | Offizielle API-Dokumentation |
| `https://github.com/everii-group/mocoapp-api-docs` | API-Docs Repo — Issues enthalten Workarounds |
| `https://www.mocoapp.com/agentursoftware/funktionen/inhaltsverzeichnis` | Feature-Dokumentation |

### Search strategy

1. **Use `web_fetch`** für bekannte Doku-URLs
2. **Use `web_search`** mit:
   - `site:everii-group.github.io/mocoapp-api-docs <topic>`
   - `mocoapp API <topic>`
   - `moco n8n integration <topic>`

---

## API Basics

### Base URL
```
https://{domain}.mocoapp.com/api/v1/
```
`{domain}` = der Subdomain-Teil eures MOCO-Accounts.

### Authentifizierung

Zwei Key-Typen:
- **User API Key** — Benutzerprofil → Tab "Integrationen"
- **Account API Key** — Einstellungen → Erweiterungen → API & Webhooks (read-only oder full access)

```
Authorization: Token token=YOUR_API_KEY
```
oder:
```
Authorization: Bearer YOUR_API_KEY
```

**Programmatisch Key anfordern:**
```
POST /api/v1/session  { "email": "...", "password": "..." }
GET  /api/v1/session  → validiert Key (200 mit User-Info oder 401)
```

**Impersonation** (Staff-Level):
```
X-IMPERSONATE-USER-ID: 123
```

### Content-Type
Alle Payloads als JSON mit `Content-Type: application/json`.

### Rate Limits

| Plan | Limit |
|------|-------|
| Standard | ~120 Requests / 2 Min (~1/s) |
| Unlimited | ~1.200 Requests / 2 Min (~10/s) |

Überschreitung → `429 Too Many Requests`.

### Pagination

Default: 100 Einträge pro Seite. Response-Headers:
- `X-Page` — aktuelle Seite
- `X-Per-Page` — Einträge pro Seite
- `X-Total` — Gesamtanzahl
- `Link` — URL zur nächsten Seite

### Sortierung
```
?sort_by=field_name        # aufsteigend (default)
?sort_by=field_name desc   # absteigend
```

### Globale Filter (alle List-Endpoints)
- `?ids=123,456` — kommaseparierte IDs
- `?updated_after=2024-01-01T00:00:00Z` — Sync-Filter
- `?custom_properties[Feld]=Wert` — Eigene Felder filtern

### HTTP Status Codes

| Code | Bedeutung |
|------|-----------|
| 200 | OK |
| 201 | Created |
| 204 | No Content |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 422 | Unprocessable Entity |
| 429 | Too Many Requests |

---

## Wichtigste API-Endpunkte (Kurzreferenz)

### Activities (Zeiteinträge)
```
GET    /activities              — Liste (Filter: from, to, user_id, project_id, task_id, billable, billed)
POST   /activities              — Erstellen (required: date, project_id, task_id)
POST   /activities/bulk         — Massenanlage
PATCH  /activities/{id}/start_timer
PATCH  /activities/{id}/stop_timer
POST   /activities/disregard    — Als bereits abgerechnet markieren
```

### Projects (Projekte)
```
GET    /projects                — Liste (Filter: include_archived, leader_id, company_id, tags)
GET    /projects/assigned       — Dem aktuellen User zugewiesen
POST   /projects                — Erstellen (required: name, currency, start_date, finish_date, fixed_price, retainer, leader_id, customer_id)
GET    /projects/{id}/report    — Business-Kennzahlen
PUT    /projects/{id}/archive
PUT    /projects/{id}/unarchive
```

### Project Tasks (Leistungen)
```
GET    /projects/{id}/tasks
POST   /projects/{id}/tasks     — Erstellen (required: name)
```

### Companies (Firmen)
```
GET    /companies               — Liste (Filter: type, tags, term)
POST   /companies               — Erstellen (required: name, type; customer braucht currency)
PUT    /companies/{id}/archive
```
Types: `customer`, `supplier`, `organization`

### Invoices (Rechnungen)
```
GET    /invoices                — Liste (Filter: status, date_from/to, company_id, project_id)
GET    /invoices/{id}.pdf       — PDF-Download
POST   /invoices                — Erstellen (required: customer_id, recipient_address, date, due_date, title, tax, currency, items)
PUT    /invoices/{id}/update_status  — Status ändern (created, sent, overdue, ignored)
POST   /invoices/{id}/send_email    — Per E-Mail versenden
```

### Users (Personal)
```
GET    /users                   — Liste
GET    /users/{id}/performance_report  — Soll-Ist (param: year)
```

### Presences (Arbeitszeiten)
```
GET    /users/presences         — Liste (Filter: from, to, user_id)
POST   /users/presences         — Erstellen (required: date, from)
POST   /users/presences/touch   — Stempeln (Ein/Aus Toggle)
```

### Deals / Leads
```
GET    /deals                   — Liste (Filter: status, tags, company_id)
POST   /deals                   — Erstellen (required: name, currency, money, reminder_date, user_id, deal_category_id)
```

### Planning Entries (Planung)
```
GET    /planning_entries        — Liste (Filter: period "YYYY-MM-DD:YYYY-MM-DD", user_id, project_id)
POST   /planning_entries        — Erstellen (required: project_id OR deal_id, starts_on, ends_on, hours_per_day)
```

### Offers (Angebote)
```
GET    /offers                  — Liste
GET    /offers/{id}.pdf         — PDF
POST   /offers                  — Erstellen
PUT    /offers/{id}/update_status
```

### Purchases (Ausgaben)
```
GET    /purchases               — Liste
POST   /purchases               — Erstellen (required: date, currency, payment_method, items)
POST   /purchases/{id}/assign_to_project
```

→ Vollständige Referenz aller Endpunkte: siehe `references/api.md`

---

## Webhooks

### Setup
```
POST /api/v1/account/web_hooks
{
  "target": "Activity",
  "event": "create",
  "hook": "https://your-endpoint.example.com/webhook"
}
```

### Targets & Events

| Target | Events |
|--------|--------|
| Activity | create, update, delete |
| Company | create, update, delete |
| Contact | create, update, delete |
| Project | create, update, delete |
| Invoice | create, update, delete |
| Offer | create, update, delete |
| Deal | create, update, delete |
| Expense | create, update, delete |

### Webhook-Headers
- `X-Moco-Target` — z.B. "Activity"
- `X-Moco-Event` — z.B. "create"
- `X-Moco-Timestamp` — Unix Timestamp
- `X-Moco-Signature` — HMAC SHA256 zur Verifizierung
- `X-Moco-User-Id` — User der die Aktion ausgelöst hat
- `X-Moco-Account-Url` — Account-URL

### Wichtig
- Muss innerhalb 10 Sekunden mit 2XX antworten
- Nach 500 aufeinanderfolgenden Fehlern wird der Webhook **automatisch deaktiviert**
- Signaturprüfung via HMAC SHA256

### Verwaltung
```
GET    /account/web_hooks
PUT    /account/web_hooks/{id}/enable
PUT    /account/web_hooks/{id}/disable
DELETE /account/web_hooks/{id}
```

---

## n8n-Integration

### HTTP Request Node — Grundsetup

**Credentials:** Generic Header Auth
- Header Name: `Authorization`
- Header Value: `Token token=DEIN_API_KEY`

**Basis-URL in jedem HTTP Request Node:**
```
https://{domain}.mocoapp.com/api/v1/{endpoint}
```

### Typische n8n-Workflows

#### Zeiteinträge auslesen
```
Method: GET
URL: https://{domain}.mocoapp.com/api/v1/activities
Query Parameters:
  from: {{ $today.format('YYYY-MM-DD') }}
  to: {{ $today.format('YYYY-MM-DD') }}
  user_id: 123
```

#### Zeiteintrag erstellen
```
Method: POST
URL: https://{domain}.mocoapp.com/api/v1/activities
Headers: Content-Type: application/json
Body (JSON):
{
  "date": "{{ $today.format('YYYY-MM-DD') }}",
  "project_id": 456,
  "task_id": 789,
  "seconds": 3600,
  "description": "Entwicklung Feature X"
}
```

#### Stempeln (Ein/Aus)
```
Method: POST
URL: https://{domain}.mocoapp.com/api/v1/users/presences/touch
```

#### Webhook-Trigger in n8n
1. n8n Webhook Node erstellen → URL kopieren
2. In MOCO: Webhook anlegen mit der n8n-URL als `hook`
3. n8n Workflow verarbeitet eingehende Events

### Pagination in n8n
Für große Datenmengen den **Loop Over Items** Node nutzen:
1. Erste Seite abrufen → `X-Total` Header auslesen
2. Schleife über `?page=2`, `?page=3`, etc.
3. Ergebnisse zusammenführen

### Rate Limiting in n8n
- **Wait Node** zwischen Requests: 1 Sekunde (Standard-Plan) oder 100ms (Unlimited)
- Alternativ: **Batch** Node um Requests zu gruppieren

---

## MCP Server

MOCO bietet einen offiziellen MCP Server an, der in Claude Code / Claude Desktop verwendet werden kann. Damit können MOCO-Daten direkt aus Claude heraus abgefragt und bearbeitet werden.

### Verfügbare MCP Tools (Beispiele)

Die MOCO MCP Tools folgen dem Muster der API-Endpunkte:
- `find_activities` — Zeiteinträge suchen
- `create_activity` — Zeiteintrag erstellen
- `find_projects` — Projekte suchen/filtern
- `find_companies` — Firmen suchen
- `find_invoices` — Rechnungen suchen
- `create_presence` / `touch_presence` — Arbeitszeiten stempeln
- `list_users` — Benutzer auflisten
- `find_presences` — Arbeitszeiten abfragen

### Einrichtung

Der MCP Server wird über die Claude Desktop Config oder Claude Code Settings eingebunden. Die Konfiguration benötigt:
- MOCO Domain (Subdomain)
- API Token

---

## MOCO Funktionsbereiche

| Bereich | Beschreibung |
|---------|-------------|
| **Projekte** | Kundenprojekte, interne Projekte, Fixpreis, Retainer, Stundenkontingent |
| **Zeiterfassung** | Projekt-Zeiterfassung, Arbeitszeiterfassung, Stoppuhr, Mobile App |
| **Abrechnung** | Rechnungen, Teilrechnungen, Sammelrechnungen, Mahnungen, Storno |
| **Personal** | Mitarbeiter, Teams, Soll-Ist, Urlaub, Abwesenheiten |
| **Leads** | Akquise-Pipeline, Forecast, Angebote |
| **Angebote** | Erstellung, Kalkulation, Vorlagen, Kundenfreigabe |
| **Planung** | Ressourcenplanung, Auslastung, Abwesenheiten |
| **Ausgaben** | Eingangsrechnungen, Spesen, Fixkosten, Budgets |
| **Firmen & Kontakte** | Kunden, Lieferanten, Kontaktpersonen |
| **Berichte** | Dashboard, Finanzen, Cashflow, Auslastung, Rentabilität |

→ Detaillierte Feature-Beschreibungen: siehe `references/features.md`

---

## Troubleshooting

| Symptom | Lösung |
|---------|--------|
| 401 Unauthorized | API Key prüfen — User Key oder Account Key? Richtige Domain? |
| 403 Forbidden | Account Key hat evtl. nur read-only Zugriff |
| 422 Unprocessable Entity | Pflichtfelder fehlen oder ungültige Werte — Response-Body enthält Details |
| 429 Too Many Requests | Rate Limit erreicht — Wait/Delay einbauen |
| Webhook deaktiviert | Nach 500 Fehlern auto-disabled → `PUT .../enable` |
| Pagination unvollständig | `X-Total` Header prüfen und alle Seiten abrufen |
| Zeiteintrag lässt sich nicht löschen | Bereits abgerechnet oder gesperrt |
| Projekt lässt sich nicht löschen | Hat noch Zeiteinträge, Rechnungen oder Angebote |
