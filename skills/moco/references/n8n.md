# MOCO + n8n Integration — Referenz

## Grundsetup

### Credentials einrichten

In n8n: **Credentials → Add Credential → Header Auth**

| Feld | Wert |
|------|------|
| Header Name | `Authorization` |
| Header Value | `Token token=DEIN_MOCO_API_KEY` |

Alternativ: **Bearer Auth** mit `DEIN_MOCO_API_KEY` als Token.

### Basis-URL
```
https://{domain}.mocoapp.com/api/v1/
```

---

## HTTP Request Node — Konfiguration

| Einstellung | Wert |
|------------|------|
| Method | GET / POST / PUT / PATCH / DELETE |
| URL | `https://{domain}.mocoapp.com/api/v1/{endpoint}` |
| Authentication | Header Auth (oder Bearer) |
| Content-Type | application/json |
| Response Format | JSON |

---

## Häufige Workflows

### 1. Tägliche Zeiterfassung abfragen

**Trigger:** Schedule Node (täglich 18:00)
**HTTP Request:**
```
GET https://{domain}.mocoapp.com/api/v1/activities
Query: from={{ $today.format('YYYY-MM-DD') }}&to={{ $today.format('YYYY-MM-DD') }}
```

### 2. Zeiteintrag erstellen

**HTTP Request:**
```
POST https://{domain}.mocoapp.com/api/v1/activities
Body:
{
  "date": "2024-03-21",
  "project_id": 456,
  "task_id": 789,
  "seconds": 3600,
  "description": "Feature-Entwicklung"
}
```

### 3. Stempeln (Ein/Aus)

**HTTP Request:**
```
POST https://{domain}.mocoapp.com/api/v1/users/presences/touch
Body: {}
```
Optional mit Homeoffice:
```json
{ "is_home_office": true }
```

### 4. Neue Rechnung erstellen

**HTTP Request:**
```
POST https://{domain}.mocoapp.com/api/v1/invoices
Body:
{
  "customer_id": 123,
  "recipient_address": "Firma GmbH\nMusterstr. 1\n12345 Berlin",
  "date": "2024-03-21",
  "due_date": "2024-04-21",
  "title": "Rechnung März 2024",
  "tax": 19.0,
  "currency": "EUR",
  "items": [
    {
      "type": "item",
      "title": "Entwicklung",
      "quantity": 40,
      "unit": "Stunden",
      "unit_price": 120.0
    }
  ]
}
```

### 5. Projekt-Report abrufen

```
GET https://{domain}.mocoapp.com/api/v1/projects/{project_id}/report
```

### 6. Webhook-Empfang in n8n

1. **Webhook Node** erstellen → URL kopieren
2. In MOCO Webhook anlegen:
   ```
   POST /api/v1/account/web_hooks
   {
     "target": "Invoice",
     "event": "create",
     "hook": "https://n8n.example.com/webhook/abc123"
   }
   ```
3. n8n Workflow verarbeitet eingehende Events

### 7. Alle Projekte mit Pagination abrufen

```
Schritt 1: HTTP Request → GET /projects?page=1
Schritt 2: Response Header "X-Total" auslesen
Schritt 3: Loop Node → Seiten 2 bis ceil(X-Total / 100)
Schritt 4: Merge Node → Alle Ergebnisse zusammenführen
```

### 8. Monatsabrechnung automatisieren

**Trigger:** Schedule Node (1. des Monats, 09:00)
```
1. GET /projects?retainer=true → aktive Retainer-Projekte
2. Für jedes Projekt: GET /activities?from=VORMONAT-01&to=VORMONAT-LETZTER&project_id=X
3. Stunden summieren
4. POST /invoices → Rechnung erstellen
5. POST /invoices/{id}/send_email → Versenden
```

---

## Pagination-Pattern in n8n

### Variante 1: HTTP Request Node mit Pagination

Ab n8n v1.x hat der HTTP Request Node eingebaute Pagination:
- **Pagination Type:** Response Contains Next URL
- **Next URL:** aus dem `Link` Response-Header

### Variante 2: Manuell mit Loop

```
1. Set Variable: page = 1, allItems = []
2. HTTP Request: GET /endpoint?page={{ page }}
3. Response Header X-Total auslesen
4. Items zu allItems hinzufügen
5. IF page * 100 < X-Total → page++ → zurück zu Schritt 2
6. ELSE → weiter mit allItems
```

---

## Rate Limiting in n8n

### Standard-Plan (~1 Request/Sekunde)
- **Wait Node** (1000ms) zwischen Requests in Loops
- Oder: **Batch Size** auf 1 setzen mit Delay

### Unlimited Plan (~10 Requests/Sekunde)
- **Wait Node** (100ms) oder kein Delay nötig bei moderater Last

### 429 Handling
```
IF Status Code = 429
  → Wait Node (30 Sekunden)
  → Retry HTTP Request
```

---

## Impersonation in n8n

Für Workflows die im Namen eines anderen Users agieren:

Zusätzlicher Header im HTTP Request Node:
```
X-IMPERSONATE-USER-ID: 123
```

⚠️ Erfordert Staff-Level API Key.

---

## Environment Variables

Empfohlen in n8n:
```
MOCO_DOMAIN=firmname
MOCO_API_TOKEN=abc123...
```

Verwendung in Nodes:
```
https://{{ $env.MOCO_DOMAIN }}.mocoapp.com/api/v1/...
```

---

## Fehlerbehandlung

| Fehler | Lösung in n8n |
|--------|--------------|
| 401 | Credentials prüfen — richtiger API Key? |
| 403 | Account Key evtl. read-only |
| 422 | Error Handling Node → Response Body enthält Details |
| 429 | Wait + Retry Pattern |
| Timeout | HTTP Request Timeout erhöhen (default 60s) |

### Error Workflow Pattern
```
1. HTTP Request Node → Continue on Fail = true
2. IF Node → Status Code ≠ 200
3. → Error Branch: Slack/Email Notification
4. → Success Branch: Weiter verarbeiten
```
