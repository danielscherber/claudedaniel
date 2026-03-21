# MOCO API v1 — Vollständige Endpunkt-Referenz

Base URL: `https://{domain}.mocoapp.com/api/v1/`

---

## Profile
| Method | Endpoint | Beschreibung |
|--------|----------|-------------|
| GET | `/profile` | Aktueller Benutzer (oder impersonierter User) |

---

## Activities (Zeiteinträge)
| Method | Endpoint | Beschreibung |
|--------|----------|-------------|
| GET | `/activities` | Liste (Filter: from, to, user_id, project_id, task_id, company_id, billable, billed, term) |
| GET | `/activities/{id}` | Einzeln |
| POST | `/activities` | Erstellen (required: date, project_id, task_id; optional: seconds, description, billable, tag, remote_service, remote_id, remote_url) |
| POST | `/activities/bulk` | Massenanlage |
| PUT | `/activities/{id}` | Aktualisieren |
| PATCH | `/activities/{id}/start_timer` | Timer starten (nur aktueller Tag) |
| PATCH | `/activities/{id}/stop_timer` | Timer stoppen |
| DELETE | `/activities/{id}` | Löschen (nur wenn nicht abgerechnet/gesperrt) |
| POST | `/activities/disregard` | Als abgerechnet markieren (required: reason, activity_ids, company_id) |

Remote Services: trello, jira, asana, basecamp, basecamp2, basecamp3, wunderlist, toggl, mite, github, youtrack.

---

## Projects (Projekte)
| Method | Endpoint | Beschreibung |
|--------|----------|-------------|
| GET | `/projects` | Liste (Filter: include_archived, include_company, leader_id, company_id, project_group_id, deal_id, created_from/to, updated_from/to, tags, identifier, retainer) |
| GET | `/projects/assigned` | Dem aktuellen User zugewiesen (Filter: active) |
| GET | `/projects/{id}` | Einzeln |
| POST | `/projects` | Erstellen (required: name, currency, start_date, finish_date, fixed_price, retainer, leader_id, customer_id) |
| PUT | `/projects/{id}` | Aktualisieren (currency nicht änderbar) |
| DELETE | `/projects/{id}` | Löschen (nur wenn keine Activities/Invoices/Offers/Expenses) |
| PUT | `/projects/{id}/archive` | Archivieren |
| PUT | `/projects/{id}/unarchive` | Dearchivieren |
| GET | `/projects/{id}/report` | Business-Kennzahlen (Budget, Stunden, Kosten) |
| PUT | `/projects/{id}/share` | Report-Sharing aktivieren (gibt URL zurück) |
| PUT | `/projects/{id}/disable_share` | Sharing deaktivieren |
| PUT | `/projects/{id}/assign_project_group` | Gruppe zuweisen |
| PUT | `/projects/{id}/unassign_project_group` | Gruppe entfernen |

Billing Variants: `project`, `task`, `user`. Retainer-Projekte: Start am 1., Ende am Letzten des Monats + budget_monthly.

---

## Project Tasks (Leistungen)
| Method | Endpoint | Beschreibung |
|--------|----------|-------------|
| GET | `/projects/{id}/tasks` | Alle Tasks eines Projekts |
| GET | `/projects/{id}/tasks/{id}` | Einzeln |
| POST | `/projects/{id}/tasks` | Erstellen (required: name; optional: billable, budget, hourly_rate, description) |
| PUT | `/projects/{id}/tasks/{id}` | Aktualisieren |
| DELETE | `/projects/{id}/tasks/{id}` | Löschen (nur wenn keine Zeiteinträge) |
| DELETE | `/projects/{id}/tasks/destroy_all` | Alle löschbaren Tasks entfernen |

---

## Project Contracts (Mitarbeiterzuweisungen)
| Method | Endpoint | Beschreibung |
|--------|----------|-------------|
| GET | `/projects/{id}/contracts` | Liste |
| GET | `/projects/{id}/contracts/{id}` | Einzeln |
| POST | `/projects/{id}/contracts` | Erstellen (required: user_id; optional: billable, active, budget, hourly_rate) |
| PUT | `/projects/{id}/contracts/{id}` | Aktualisieren |
| DELETE | `/projects/{id}/contracts/{id}` | Löschen (fehlschlag wenn Stunden gebucht) |

---

## Project Expenses (Zusatzleistungen)
| Method | Endpoint | Beschreibung |
|--------|----------|-------------|
| GET | `/projects/{id}/expenses` | Pro Projekt (Filter: billable, billed, budget_relevant, user_id) |
| GET | `/projects/expenses` | Alle Projekte (Filter: ids, updated_after, from, to, billable, billed, budget_relevant, tags, user_id) |
| GET | `/projects/{id}/expenses/{id}` | Einzeln |
| POST | `/projects/{id}/expenses` | Erstellen (required: date, title, quantity, unit, unit_price, unit_cost) |
| POST | `/projects/{id}/expenses/bulk` | Massenanlage |
| PUT | `/projects/{id}/expenses/{id}` | Aktualisieren (nur wenn nicht abgerechnet) |
| DELETE | `/projects/{id}/expenses/{id}` | Löschen (nur wenn nicht abgerechnet) |
| POST | `/projects/{id}/expenses/disregard` | Als abgerechnet markieren |

---

## Project Recurring Expenses
| Method | Endpoint | Beschreibung |
|--------|----------|-------------|
| GET | `/recurring_expenses` | Alle |
| GET | `/projects/{id}/recurring_expenses` | Pro Projekt |
| GET | `/projects/{id}/recurring_expenses/{id}` | Einzeln |
| POST | `/projects/{id}/recurring_expenses` | Erstellen (required: start_date, period, title, quantity, unit, unit_price, unit_cost) |
| PUT | `/projects/{id}/recurring_expenses/{id}` | Aktualisieren (start_date/period nicht änderbar) |
| POST | `/projects/{id}/recurring_expenses/{id}/recur` | Manuell nächste Wiederholung auslösen |
| DELETE | `/projects/{id}/recurring_expenses/{id}` | Löschen |

Perioden: weekly, biweekly, monthly, quarterly, biannual, annual.

---

## Project Payment Schedules
| Method | Endpoint | Beschreibung |
|--------|----------|-------------|
| GET | `/projects/payment_schedules` | Alle (Filter: from, to, checked, company_id, project_id) |
| GET | `/projects/{project_id}/payment_schedules` | Pro Projekt |
| POST | `/projects/{project_id}/payment_schedules` | Erstellen (required: net_total, date) |
| PUT | `/projects/{project_id}/payment_schedules/{id}` | Aktualisieren |
| DELETE | `/projects/{project_id}/payment_schedules/{id}` | Löschen |

---

## Project Groups
| Method | Endpoint | Beschreibung |
|--------|----------|-------------|
| GET | `/projects/groups` | Liste (Filter: user_id, company_id) |
| GET | `/projects/groups/{id}` | Einzeln |

Zuweisung erfolgt über den Projects-Endpoint.

---

## Companies (Firmen)
| Method | Endpoint | Beschreibung |
|--------|----------|-------------|
| GET | `/companies` | Liste (Filter: include_archived, type, tags, identifier, term) |
| GET | `/companies/{id}` | Einzeln |
| POST | `/companies` | Erstellen (required: name, type; customer braucht currency) |
| PUT | `/companies/{id}` | Aktualisieren |
| DELETE | `/companies/{id}` | Löschen |
| PUT | `/companies/{id}/archive` | Archivieren |
| PUT | `/companies/{id}/unarchive` | Dearchivieren |

Types: `customer`, `supplier`, `organization`. Invoice Formats: `regular_pdf`, `x_invoice`, `zugferd_x_invoice`.

---

## Contacts (Kontakte)
| Method | Endpoint | Beschreibung |
|--------|----------|-------------|
| GET | `/contacts/people` | Liste (Filter: tags, term, phone) |
| GET | `/contacts/people/{id}` | Einzeln |
| POST | `/contacts/people` | Erstellen (required: lastname, gender) |
| PUT | `/contacts/people/{id}` | Aktualisieren |
| DELETE | `/contacts/people/{id}` | Löschen |

---

## Deals / Leads
| Method | Endpoint | Beschreibung |
|--------|----------|-------------|
| GET | `/deals` | Liste (Filter: status, tags, closed_from/to, company_id) |
| GET | `/deals/{id}` | Einzeln |
| POST | `/deals` | Erstellen (required: name, currency, money, reminder_date, user_id, deal_category_id) |
| PUT | `/deals/{id}` | Aktualisieren |
| DELETE | `/deals/{id}` | Löschen |

Statuses: potential, pending, won, lost, dropped.

---

## Deal Categories (Akquise-Stufen)
| Method | Endpoint | Beschreibung |
|--------|----------|-------------|
| GET | `/deal_categories` | Liste |
| POST | `/deal_categories` | Erstellen (required: name, probability) |
| PUT | `/deal_categories/{id}` | Aktualisieren |
| DELETE | `/deal_categories/{id}` | Löschen (fehlschlag wenn in Verwendung) |

---

## Invoices (Rechnungen)
| Method | Endpoint | Beschreibung |
|--------|----------|-------------|
| GET | `/invoices` | Liste (Filter: status, date_from/to, service_period_from/to, tags, identifier, term, company_id, project_id, include_disregarded, not_booked) |
| GET | `/invoices/locked` | Gesperrte Rechnungen |
| GET | `/invoices/{id}` | Einzeln (inkl. Items, Payments, Reminders) |
| GET | `/invoices/{id}.pdf` | PDF (optional: blank, letter_paper_id) |
| GET | `/invoices/{id}/timesheet` | Abgerechnete Zeiteinträge |
| GET | `/invoices/{id}/timesheet.pdf` | Stundenauszug PDF |
| GET | `/invoices/{id}/expenses` | Abgerechnete Zusatzleistungen |
| POST | `/invoices` | Erstellen (required: customer_id, recipient_address, date, due_date, title, tax, currency, items) |
| PUT | `/invoices/{id}/update_status` | Status ändern (created, sent, overdue, ignored) |
| POST | `/invoices/{id}/send_email` | Per E-Mail senden (required: subject, text) |
| DELETE | `/invoices/{id}` | Löschen (required: reason, außer Entwürfe) |
| GET | `/invoices/{id}/attachments` | Anhänge auflisten |
| POST | `/invoices/{id}/attachments` | PDF-Anhang hinzufügen (base64) |
| DELETE | `/invoices/{id}/attachments/{id}` | Anhang entfernen |

Item Types: title, description, item, subtotal, page-break, separator.
Statuses: draft, created, sent, partially_paid, paid, overdue, ignored.

Template-Variablen in Anrede/Footer: `{salutation}`, `{date}`, `{due_date}`, `{recipient}`, `{company_name}`, `{net_total}`, `{tax}`, `{gross_total}`, `{title}`, `{identifier}`, `{me}`, `{user}`, `{service_period}`, `{cash_discount}`, `{signature_image}`.

---

## Invoice Payments (Zahlungen)
| Method | Endpoint | Beschreibung |
|--------|----------|-------------|
| GET | `/invoices/payments` | Liste (Filter: invoice_id, date_from/to) |
| POST | `/invoices/payments` | Erstellen (required: date, paid_total) |
| POST | `/invoices/payments/bulk` | Massenanlage |
| PUT | `/invoices/payments/{id}` | Aktualisieren (invoice_id nicht änderbar) |
| DELETE | `/invoices/payments/{id}` | Löschen |

---

## Invoice Reminders (Mahnungen)
| Method | Endpoint | Beschreibung |
|--------|----------|-------------|
| GET | `/invoice_reminders` | Liste (Filter: invoice_id, date_from/to) |
| POST | `/invoice_reminders` | Erstellen (required: invoice_id) |
| DELETE | `/invoice_reminders/{id}` | Löschen |
| POST | `/invoice_reminders/{id}/send_email` | Per E-Mail senden |

---

## Invoice Bookkeeping Exports
| Method | Endpoint | Beschreibung |
|--------|----------|-------------|
| GET | `/invoices/bookkeeping_exports` | Liste |
| POST | `/invoices/bookkeeping_exports` | Erstellen (required: invoice_ids) |

---

## Offers (Angebote)
| Method | Endpoint | Beschreibung |
|--------|----------|-------------|
| GET | `/offers` | Liste (Filter: status, from/to, identifier, deal_id, project_id, company_id) |
| GET | `/offers/{id}` | Einzeln (inkl. Items) |
| GET | `/offers/{id}.pdf` | PDF |
| POST | `/offers` | Erstellen (required: recipient_address, date, due_date, title, tax, items) |
| PUT | `/offers/{id}/assign` | Firma/Projekt/Deal zuweisen |
| PUT | `/offers/{id}/update_status` | Status ändern (created, sent, accepted, partially_billed, billed, archived) |
| POST | `/offers/{id}/send_email` | Per E-Mail senden |

Offer Customer Approval:
| GET | `/offers/{id}/customer_approval` | Status (404 wenn nicht aktiviert) |
| POST | `/offers/{id}/customer_approval/activate` | Aktivieren |
| POST | `/offers/{id}/customer_approval/deactivate` | Deaktivieren |

---

## Purchases (Ausgaben)
| Method | Endpoint | Beschreibung |
|--------|----------|-------------|
| GET | `/purchases` | Liste (Filter: category_id, company_id, status, tags, Datumsbereich, payment_method, identifier, term) |
| GET | `/purchases/{id}` | Einzeln |
| POST | `/purchases` | Erstellen (required: date, currency, payment_method, items) |
| PUT | `/purchases/{id}` | Aktualisieren (Items-Update NICHT unterstützt) |
| PATCH | `/purchases/{id}/update_status` | Status ändern (pending/archived) |
| PATCH | `/purchases/{id}/store_document` | Dokument hochladen (multipart/form-data) |
| POST | `/purchases/{id}/assign_to_project` | Projekt-Expense zuweisen |
| DELETE | `/purchases/{id}` | Löschen (nur wenn pending + keine Zahlungen) |

Payment Methods: bank_transfer, direct_debit, credit_card, paypal, cash, bank_transfer_swiss_qr_esr.

---

## Purchase Payments
| Method | Endpoint | Beschreibung |
|--------|----------|-------------|
| GET | `/purchases/payments` | Liste |
| POST | `/purchases/payments` | Erstellen (required: date, total) |
| POST | `/purchases/payments/bulk` | Massenanlage |

---

## Purchase Categories
| Method | Endpoint | Beschreibung |
|--------|----------|-------------|
| GET | `/purchases/categories` | Liste (read-only) |

---

## Purchase Drafts (Entwürfe)
| Method | Endpoint | Beschreibung |
|--------|----------|-------------|
| GET | `/purchases/drafts` | Liste |
| GET | `/purchases/drafts/{id}.pdf` | PDF (204 wenn nicht verfügbar) |
| DELETE | `/purchases/drafts/{id}` | Löschen |

---

## Purchase Bookkeeping Exports
| Method | Endpoint | Beschreibung |
|--------|----------|-------------|
| GET | `/purchases/bookkeeping_exports` | Liste |
| POST | `/purchases/bookkeeping_exports` | Erstellen (required: purchase_ids) |

---

## Purchase Budgets
| Method | Endpoint | Beschreibung |
|--------|----------|-------------|
| GET | `/purchases/budgets` | Liste (required: year) — read-only |

---

## Receipts (Spesen)
| Method | Endpoint | Beschreibung |
|--------|----------|-------------|
| GET | `/receipts` | Liste (Filter: from/to, project_id, user_id, purchase_category_id, refund_request_id, submitted) |
| POST | `/receipts` | Erstellen (required: date, title, currency, items mit vat_code_id + gross_total, purchase_category_id) |
| PATCH | `/receipts/{id}` | Aktualisieren (items werden komplett überschrieben) |
| DELETE | `/receipts/{id}` | Löschen (nur vor Refund Request) |

---

## Users (Personal)
| Method | Endpoint | Beschreibung |
|--------|----------|-------------|
| GET | `/users` | Liste (Filter: include_archived, tags, email) |
| GET | `/users/{id}` | Einzeln |
| POST | `/users` | Erstellen (required: firstname, lastname, email, unit_id) |
| PUT | `/users/{id}` | Aktualisieren |
| DELETE | `/users/{id}` | Löschen (nur wenn keine abrechnungsfähigen Stunden) |
| GET | `/users/{id}/performance_report` | Soll-Ist (param: year) |

Sprachen: de, de-AT, de-CH, en, it, fr. Avatar via base64 Upload.

---

## User Roles (Benutzerrollen)
| Method | Endpoint | Beschreibung |
|--------|----------|-------------|
| GET | `/users/roles` | Liste aller Rollen mit Berechtigungen (read-only) |

---

## Employments (Wochenmodell)
| Method | Endpoint | Beschreibung |
|--------|----------|-------------|
| GET | `/users/employments` | Liste (Filter: from, to, user_id) |
| POST | `/users/employments` | Erstellen (required: user_id, pattern) |
| PUT | `/users/employments/{id}` | Aktualisieren |
| DELETE | `/users/employments/{id}` | Löschen |

Pattern: `{"am": [h,h,h,h,h], "pm": [h,h,h,h,h]}` für Mo-Fr.

---

## Holidays (Urlaubsanspruch)
| Method | Endpoint | Beschreibung |
|--------|----------|-------------|
| GET | `/users/holidays` | Liste (Filter: year, user_id) |
| POST | `/users/holidays` | Erstellen (required: year, title, days, user_id) |
| PUT | `/users/holidays/{id}` | Aktualisieren |
| DELETE | `/users/holidays/{id}` | Löschen |

---

## Presences (Arbeitszeiten)
| Method | Endpoint | Beschreibung |
|--------|----------|-------------|
| GET | `/users/presences` | Liste (Filter: from/to, user_id, is_home_office) |
| GET | `/users/presences/{id}` | Einzeln |
| POST | `/users/presences` | Erstellen (required: date, from; optional: to, is_home_office) |
| POST | `/users/presences/touch` | Stempeln Ein/Aus (optional: override "YYYY-MM-DD HH:MM") |
| PUT | `/users/presences/{id}` | Aktualisieren |
| DELETE | `/users/presences/{id}` | Löschen |

Touch innerhalb derselben Minute wird ignoriert. 423 bei Konflikten.

---

## Work Time Adjustments (Korrekturen)
| Method | Endpoint | Beschreibung |
|--------|----------|-------------|
| GET | `/users/work_time_adjustments` | Liste (Filter: from, to, user_id) |
| POST | `/users/work_time_adjustments` | Erstellen (required: user_id, description, date, hours — positiv oder negativ) |
| PUT | `/users/work_time_adjustments/{id}` | Aktualisieren |
| DELETE | `/users/work_time_adjustments/{id}` | Löschen |

---

## Planning Entries (Planung)
| Method | Endpoint | Beschreibung |
|--------|----------|-------------|
| GET | `/planning_entries` | Liste (Filter: period "YYYY-MM-DD:YYYY-MM-DD", user_id, project_id, deal_id) |
| GET | `/planning_entries/{id}` | Einzeln |
| POST | `/planning_entries` | Erstellen (required: project_id ODER deal_id, starts_on, ends_on, hours_per_day; optional: user_id, comment, symbol 1-10, tentative) |
| PUT | `/planning_entries/{id}` | Aktualisieren |
| DELETE | `/planning_entries/{id}` | Löschen |

---

## Schedules / Absences (Absenzen)
| Method | Endpoint | Beschreibung |
|--------|----------|-------------|
| GET | `/schedules` | Liste (Filter: from, to, user_id, absence_code, holiday_request_id) |
| POST | `/schedules` | Erstellen (required: date, absence_code; optional: user_id, am, pm, comment, symbol 1-12, overwrite) |
| PUT | `/schedules/{id}` | Aktualisieren |
| DELETE | `/schedules/{id}` | Löschen |

Absence Codes: 1=Unplanbar, 2=Feiertag, 3=Krankheit, 4=Urlaub, 5=Abwesenheit.

---

## Comments (Notizen)
| Method | Endpoint | Beschreibung |
|--------|----------|-------------|
| GET | `/comments` | Liste (Filter: commentable_type, commentable_id, user_id, manual) |
| POST | `/comments` | Erstellen (required: commentable_id, commentable_type, text) |
| POST | `/comments/bulk` | Massenanlage |
| PUT | `/comments/{id}` | Aktualisieren |
| DELETE | `/comments/{id}` | Löschen (nur manuelle) |

Commentable Types: Company, Contact, Deal, Project, Invoice, Offer, Purchase, Receipt.
Erlaubte HTML-Tags: div, strong, em, u, pre, ul, ol, li, br.

---

## Reports (Berichte)
| Method | Endpoint | Beschreibung |
|--------|----------|-------------|
| GET | `/report/absences` | Abwesenheiten pro User (Filter: active, year) |
| GET | `/report/cashflow` | Cashflow (Filter: from, to, term) |
| GET | `/report/finance` | Finanzbericht (Filter: from, to, term) |
| GET | `/report/planned_vs_tracked` | Planung vs. Ist (Filter: from, to) |
| GET | `/report/utilization` | Tägliche Auslastung (Filter: from, to) |

---

## Units / Teams
| Method | Endpoint | Beschreibung |
|--------|----------|-------------|
| GET | `/units` | Liste |
| POST | `/units` | Erstellen (required: name) |
| PUT | `/units/{id}` | Aktualisieren |
| DELETE | `/units/{id}` | Löschen (nur wenn keine User zugewiesen) |

---

## Tags
| Method | Endpoint | Beschreibung |
|--------|----------|-------------|
| GET | `/tags` | Liste (Filter: context) |
| POST | `/tags` | Erstellen (required: name, context; optional: color als Hex) |
| PUT | `/tags/{id}` | Aktualisieren (context unveränderbar) |
| DELETE | `/tags/{id}` | Löschen (optional: merge_tag_id zum Umhängen) |

Contexts: Company, Contact, Project, Deal, Purchase, Invoice, Offer, User.

---

## Taggings (Tag-Zuweisungen)
| Method | Endpoint | Beschreibung |
|--------|----------|-------------|
| GET | `/taggings/{entity}/{entity_id}` | Tags einer Entity |
| PATCH | `/taggings/{entity}/{entity_id}` | Tags hinzufügen (bestehende bleiben) |
| PUT | `/taggings/{entity}/{entity_id}` | Alle Tags ersetzen |
| DELETE | `/taggings/{entity}/{entity_id}` | Tags entfernen |

---

## VAT Codes (Steuerschlüssel)
| Method | Endpoint | Beschreibung |
|--------|----------|-------------|
| GET | `/vat_code_sales` | Verkaufs-Steuercodes (Filter: reverse_charge, intra_eu, active) |
| GET | `/vat_code_purchases` | Einkaufs-Steuercodes |

---

## Web Hooks
| Method | Endpoint | Beschreibung |
|--------|----------|-------------|
| GET | `/account/web_hooks` | Liste |
| POST | `/account/web_hooks` | Erstellen (required: target, event, hook URL) |
| PUT | `/account/web_hooks/{id}` | Aktualisieren (nur hook-Feld) |
| PUT | `/account/web_hooks/{id}/enable` | Aktivieren |
| PUT | `/account/web_hooks/{id}/disable` | Deaktivieren |
| DELETE | `/account/web_hooks/{id}` | Löschen |

---

## Account-Einstellungen

### Custom Properties (Eigene Felder)
```
GET    /account/custom_properties        — Liste (Filter: entity)
POST   /account/custom_properties        — Erstellen (required: name, kind, entity)
PATCH  /account/custom_properties/{id}   — Aktualisieren
DELETE /account/custom_properties/{id}   — Löschen
```
Kinds: String, Textarea, Link, Boolean, Select, MultiSelect, Date.

### Hourly Rates
```
GET /account/hourly_rates               — Stundensätze (optional: company_id)
```

### Internal Hourly Rates
```
GET   /account/internal_hourly_rates    — Interne Stundensätze (params: years, unit_id)
PATCH /account/internal_hourly_rates    — Aktualisieren
```

### Fixed Costs
```
GET /account/fixed_costs                — Fixkosten (Filter: year) — read-only
```

### Catalog Services (Leistungskatalog)
```
GET    /account/catalog_services
POST   /account/catalog_services        — Mit Items-Array erstellen
POST   /account/catalog_services/{id}/items  — Items hinzufügen
PUT    /account/catalog_services/{id}   — Titel aktualisieren
DELETE /account/catalog_services/{id}   — Löschen
```

### Expense Templates (Zusatzleistungs-Katalog)
```
GET    /account/expense_templates
POST   /account/expense_templates       — Erstellen (required: title, description, unit, unit_price, unit_cost, currency)
PUT    /account/expense_templates/{id}
DELETE /account/expense_templates/{id}
```

### Task Templates (Standardleistungen)
```
GET    /account/task_templates
POST   /account/task_templates          — Erstellen (required: name)
PUT    /account/task_templates/{id}
DELETE /account/task_templates/{id}
```

---

## Custom Properties — Datenformat

Max 12 KB. Werte sind Strings, außer MultiSelect (Arrays).

| Typ | Beispiel |
|-----|---------|
| String | `"Wert"` |
| Textarea | `"Mehrzeiliger\nText"` |
| Link | `"https://example.com"` |
| Boolean | `"0"` oder `"1"` |
| Date | `"2024-12-31"` |
| Select | `"Option A"` |
| MultiSelect | `["Option A", "Option B"]` |

Filter: `?custom_properties[Feldname]=Wert`
Boolean: `?custom_properties[Flag]=1`
MultiSelect: `?custom_properties[Tags][]=A&custom_properties[Tags][]=B`
