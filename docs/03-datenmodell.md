# Book2Dance — Datenmodell & Rechte (Vorlage für die Entwicklung)

> Arbeitsstand: Entwurf v0.1 · Stand: 09.06.2026
> Zweck: Konkrete Vorlage für die Umsetzung der im Prototyp (`prototype/index.html`) gebauten Funktionen.
> Beschreibt das **Datenmodell** (Entitäten, Felder, Enums) und die **Rechte-Matrix** (Rolle × Aktion).
> Gegenstück zu `01-produktbeschreibung.md` (was) und `02-designflows.md` (Klickflow). Hier steht das **wie** für die kritischen Modelle.

Konventionen: `snake_case` Felder, IDs als `uuid`, Zeitstempel `created_at`/`updated_at` implizit. Mandanten­fähigkeit: fast jede Tabelle hängt an `school_id`; globale Agentur-Daten an `agency_id` mit `school_id = null`.

---

## 1. Entitäten-Überblick

```
Agency ──┬── School ──┬── User (Mitglied, mit Rolle)
         │            ├── Course ──── RecurrenceRule (0..1)
         │            │            └── Session (Termin-Instanz) ──── Booking (Teilnehmer)
         │            ├── Tag             (lokal, scope=school)
         │            ├── MessageTemplate (lokal, scope=school)
         │            ├── ReminderPolicy  (school-Ebene)
         │            └── Branding (Theme/Look)
         ├── Tag             (global, scope=agency)
         ├── MessageTemplate (global, scope=agency)
         └── ReminderPolicy  (agency-Ebene = Default)
```

- **Course** = der Kurs (Discofox Schnupperstunde). Trägt Stammdaten, Merkmale, Kurs-Modell.
- **RecurrenceRule** = die Wiederholungsregel (Serie). Erzeugt **Session**-Instanzen.
- **Session** = ein konkreter Termin mit Datum/Uhrzeit/Plätzen/eigener Teilnehmerliste. Einzeln editier-/absagbar.
- **Booking** = Anmeldung eines Interessenten zu **einer** Session.

---

## 2. Rollen & Rechte-Matrix

### 2.1 Rollen

| Ebene | Rolle | Beschreibung |
|---|---|---|
| Agentur | `agency_admin` | Plattform-Betreiber. Legt Schulen an, verwaltet globale Merkmale & Reminder-Defaults, sieht alles. |
| Schule | `superadmin` | Vollzugriff auf die eigene Schule inkl. Team, Branding, Einstellungen. **Mind. 1 pro Schule Pflicht.** |
| Schule | `admin` | Tagesgeschäft: Kurse, Termine, Teilnehmer. **Kein** Zugriff auf Team/Branding/Einstellungen. |
| Schule | `reader` | Nur-Lesen: sieht Kurse & Teilnehmerlisten, ändert nichts. |
| — | (anonym) | Endkunde/Interessent, kein Login. Nur öffentliche Buchung. |

> Trainer-Rolle (siehe `01-…` F3) ist Phase 2 und hier bewusst noch nicht abgebildet.

### 2.2 Matrix (Rolle × Aktion)

Legende: ✅ erlaubt · 👁 nur lesen · ❌ nicht erlaubt · — nicht zutreffend

| Aktion | agency_admin | superadmin | admin | reader |
|---|:--:|:--:|:--:|:--:|
| Schule anlegen / archivieren | ✅ | ❌ | ❌ | ❌ |
| Schulen-übergreifende Auswertung | ✅ | ❌ | ❌ | ❌ |
| Globale Merkmale verwalten | ✅ | ❌ | ❌ | ❌ |
| Reminder-Default (Agentur) setzen | ✅ | ❌ | ❌ | ❌ |
| Branding/Look der Schule ändern | ✅ | ✅ | ❌ | 👁 |
| Lokale Merkmale der Schule verwalten | ✅ | ✅ | ✅ | 👁 |
| Reminder-Standard der Schule setzen | ✅ | ✅ | ❌ | 👁 |
| Team einsehen | ✅ | ✅ | 👁 | 👁 |
| Mitglieder einladen / Rolle ändern / entfernen | ✅ | ✅ | ❌ | ❌ |
| Kurs anlegen / bearbeiten / löschen | ✅ | ✅ | ✅ | 👁 |
| Serie/Termine erzeugen & bearbeiten | ✅ | ✅ | ✅ | 👁 |
| Reminder pro Kurs überschreiben | ✅ | ✅ | ✅ | 👁 |
| Termin absagen / verschieben | ✅ | ✅ | ✅ | ❌ |
| Teilnehmerliste sehen / Status setzen / CSV-Export | ✅ | ✅ | ✅ | 👁 |
| Teilnehmer manuell eintragen | ✅ | ✅ | ✅ | ❌ |
| Öffentlich buchen (Interessent) | — | — | — | — |

### 2.3 Invarianten

- Eine Schule hat **immer mindestens einen `superadmin`** — der letzte darf sich nicht selbst herabstufen/entfernen.
- `agency_admin` kann „in eine Schule wechseln" und handelt dort mit `superadmin`-Rechten (Impersonation), sollte aber protokolliert werden (`audit_log`).
- Rollen sind **pro Schule**: dieselbe Person kann in Schule A `admin`, in Schule B `reader` sein → Zwischen­tabelle `membership`.

```
Membership { id, user_id, school_id, role: enum(superadmin|admin|reader), status: enum(active|invited|disabled), invited_at, accepted_at }
```

---

## 3. Serie / Wiederholung (RecurrenceRule, RRULE-orientiert)

Angelehnt an iCalendar **RRULE** (RFC 5545), bewusst auf die im Prototyp gebauten Fälle reduziert. Die Regel hängt am Kurs; aus ihr werden **Session**-Instanzen materialisiert.

```
RecurrenceRule {
  id
  course_id
  freq        : enum(DAILY | WEEKLY | MONTHLY)   // "Wiederholen": Täglich/Wöchentlich/Monatlich
  interval    : int >= 1                          // "alle N" (1 = jede, 2 = jede 2., …)
  by_weekday  : int[]    // nur bei WEEKLY. 0=So … 6=Sa (JS getDay). Mehrere möglich (Mo+Mi).
  month_mode  : enum(BY_MONTHDAY | BY_WEEKDAY) | null  // nur bei MONTHLY
  // BY_MONTHDAY: immer am X. Tag (X = Tag des Startdatums)
  // BY_WEEKDAY : am n-ten Wochentag (z. B. 3. Dienstag) — n & Wochentag aus Startdatum abgeleitet
  start_date  : date     // erster Termin
  time        : time     // Uhrzeit
  duration_min: int      // Dauer in Minuten
  seats       : int      // Standard-Plätze pro Termin
  end_mode    : enum(COUNT | UNTIL)
  end_count   : int  | null   // bei COUNT: Anzahl Termine
  end_until   : date | null   // bei UNTIL: bis einschließlich Datum
}
```

### 3.1 Mapping auf iCal RRULE (für spätere .ics-Exporte / Kalender-Sync)

| unser Feld | RRULE |
|---|---|
| `freq=WEEKLY, interval=2, by_weekday=[1,3]` | `FREQ=WEEKLY;INTERVAL=2;BYDAY=MO,WE` |
| `freq=MONTHLY, month_mode=BY_MONTHDAY` (Start 16.) | `FREQ=MONTHLY;BYMONTHDAY=16` |
| `freq=MONTHLY, month_mode=BY_WEEKDAY` (3. Di) | `FREQ=MONTHLY;BYDAY=3TU` |
| `end_mode=COUNT, end_count=8` | `COUNT=8` |
| `end_mode=UNTIL, end_until=2026-08-31` | `UNTIL=20260831T...` |

> Wochentag-Codes: 0=So,1=Mo,2=Di,3=Mi,4=Do,5=Fr,6=Sa (entspricht JS `Date.getDay()`; bei .ics-Export auf SU/MO/… mappen).

### 3.2 Materialisierung & Bearbeitung

- Beim Speichern der Regel werden **Session**-Instanzen erzeugt (Prototyp-Logik in `generateSeries()` ist die Referenz: Tages-/Wochen-/Monatslogik inkl. Mehrfach-Wochentagen und n-tem-Wochentag).
- Jede Session ist **eigenständig**: eigene `seats`, eigene Teilnehmerliste, eigene Termin-Merkmale.
- Eine einzeln geänderte oder abgesagte Session **löst sich von der Regel** (`detached = true`), analog zu „diesen Termin bearbeiten" im Kalender. Neu-Generieren der Regel überschreibt detachte Sessions nicht.
- Offene Designentscheidung: Wie weit im Voraus materialisieren (alle auf einmal vs. rollierendes Fenster)? Für „feste Serie" (entschieden) genügt **einmalig komplett erzeugen**.

```
Session {
  id, course_id, recurrence_rule_id | null,
  date, time, duration_min, seats,
  status: enum(planned | cancelled | done),
  detached: bool,                    // wurde individuell verändert
  features: Feature[]                // Termin-spezifische Merkmale (überschreiben/ergänzen Kurs-Merkmale)
}
```

---

## 4. Tags — global + lokal

Wiederverwendbare Chips, die auf der Buchungsseite über dem Kurstitel erscheinen (z. B. „Anfänger", „Kein Partner nötig"). Im UI **Tags** genannt; der heutige Katalog ist der **Kurs-Tags-Katalog** (Tags, die an Kurse/Termine gesetzt werden). `category` lässt Raum für weitere Tag-Kataloge später.

```
Tag {
  id
  label       : string
  category    : enum(course | …)         // aktuell nur "course"; erweiterbar
  scope       : enum(agency | school)    // agency = global für alle Schulen
  agency_id   : uuid          // immer gesetzt
  school_id   : uuid | null   // nur bei scope=school
  archived    : bool          // statt hartem Löschen, falls bereits verwendet
}
```

- **Global** (`scope=agency`): von der Agentur einmal gepflegt, stehen allen Schulen als Schnellauswahl zur Verfügung. Schule kann sie nutzen, aber **nicht ändern/löschen**.
- **Lokal** (`scope=school`): die Schule legt eigene an (in den Einstellungen).
- Zuordnung zu Kurs/Termin als n:m über `course_tag` bzw. `session_tag` (Session-Tags ergänzen die des Kurses).
- **Löschen eines verwendeten globalen Tags** → `archived=true` statt Delete; bestehende Zuordnungen bleiben erhalten, es taucht nur nicht mehr in der Schnellauswahl auf.

---

## 5. Reminder — dreistufige Vererbung (Agentur → Schule → Kurs)

Erinnerungen an gebuchte Teilnehmer vor dem Termin. Frei konfigurierbar: beliebig viele Regeln, je Regel Zeitpunkt + Kanäle.

```
ReminderRule {
  offset_value : int
  offset_unit  : enum(MIN | HOUR | DAY)   // "X vorher"
  channels     : { email:bool, sms:bool, whatsapp:bool, voice:bool }
  templates    : { email:tpl_id, sms:tpl_id, whatsapp:tpl_id, voice:tpl_id }  // je aktivem Kanal eine Vorlage (siehe §5.2)
}

ReminderPolicy {
  id
  level        : enum(agency | school | course)
  owner_id     : uuid           // agency_id | school_id | course_id je nach level
  inherit      : bool           // true = Ebene darüber verwenden (school/course), bei agency immer false
  rules        : ReminderRule[] // nur relevant wenn inherit=false
}
```

### 5.1 Auflösung der effektiven Regeln

```
effective(course)  = course.inherit  ? effective(school) : course.rules
effective(school)  = school.inherit  ? agency.rules      : school.rules
agency.rules       = immer direkt (oberste Ebene)
```

- Im Prototyp entspricht das den Schaltern „Schul-Standard verwenden" (Kurs) bzw. „Agentur-Standard verwenden" (Schule).
- Kanäle: `email`, `sms`, `whatsapp`, `voice` (automatischer Anruf).

### 5.1a Absender-Identität (Agentur + Schule, vererbt)

Wie Reminder beim Empfänger erscheinen. Existiert auf **Agentur-** und **Schul-Ebene**; die Schule kann den Agentur-Versand übernehmen oder eigenen setzen.

```
SenderIdentity {
  owner_level    : enum(agency | school)
  owner_id       : uuid
  from_name      : string   // E-Mail-Absendername
  from_email     : string   // Absende-Adresse
  email_verified : bool      // SPF/DKIM/DMARC der Domain geprüft
  reply_to       : string
  sms_sender_id  : string    // max 11 Zeichen, alphanumerisch
  wa_connected   : bool      // WhatsApp-Business-Nummer verbunden
  voice_caller_id: string | null
}
```

Auflösung: `effective_sender(school) = school.use_agency ? agency.sender : school.sender`. Beispiel: Agentur „DanceConsult" versendet über „Dancepreneur"; kleine Schulen übernehmen das, große setzen eigene Domain.

### 5.1a-2 Versand-Infrastruktur

Kein roher SMTP-Server pro Schule. Best Practice Multi-Tenant:

- **Standard (managed):** **eine zentrale Versand-Infrastruktur** auf Plattform-/Agentur-Ebene — ein Transactional-ESP (z. B. Postmark / Amazon SES / SendGrid) per API, **nicht** die Plattform-Hauptdomain. SMS/WhatsApp/Anruf laufen analog über **einen** Telefonie-Anbieter (z. B. Twilio/Bird; WhatsApp braucht ein WABA).
- **Pro Schule Domain-Authentifizierung:** jede Schule signiert ihre (Sub-)Domain per **DKIM/SPF/DMARC** → Mails kommen sichtbar von der Schule, gute Zustellbarkeit (`SenderIdentity.email_verified`).
- **Optional pro Schule (BYO):** eigener SMTP/ESP — Schule trägt Host/Port/User/Passwort/Verschlüsselung (oder ESP-API-Key) ein und verantwortet Zustellbarkeit/Reputation selbst.
- Transaktional (Reminder/Bestätigung) von Marketing trennen; Shared-IP-Pool mit Throttling, dedizierte IP erst bei Volumen.

```
EmailDelivery {                      // existiert auf Agentur- UND Schul-Ebene
  owner_level: enum(agency | school)
  mode: enum(managed | own_smtp)     // managed = Plattform-ESP + DKIM; own_smtp = eigener Server
  own_smtp: { label, host, port, username, password, encryption: STARTTLS|SSL|none } | null
}
School.use_agency_delivery: bool      // true = Agentur-Versand (z. B. Dancepreneur) übernehmen
```

Dreistufig: **Book2Dance (Plattform-ESP)** → **Agentur** (managed ODER eigener SMTP wie Dancepreneur) → **Schule** (`use_agency_delivery=true` übernimmt die Agentur, sonst eigener `managed`/`own_smtp`).

### 5.1b Einwilligung (Consent, pro Buchung)

Pro **Booking** wird festgehalten, über welche Kanäle erinnert werden darf. E-Mail ist transaktional (Default an); SMS/WhatsApp/Anruf erfordern aktives **Opt-in** und eine Telefonnummer.

```
Booking.consent { email:bool, sms:bool, whatsapp:bool, voice:bool, consented_at, source:'booking_form' }
```

- **Versand-Regel:** ein Reminder geht über Kanal X nur, wenn `effective_rule.channels[X]` **und** `booking.consent[X]` true sind (und für sms/whatsapp/voice eine Telefonnummer vorliegt).
- Widerruf jederzeit möglich (Abmelde-Link/STOP) → setzt `consent[X]=false`.

### 5.2 MessageTemplate (Nachrichten-Vorlagen)

Jeder aktive Reminder-Kanal verweist auf eine **Vorlage**. Vorlagen sind kanal­spezifisch und global (Agentur) oder lokal (Schule) — gleiche Scope-Logik wie Tags.

```
MessageTemplate {
  id
  name        : string                 // im Reminder-Workflow als erkennbarer Name angezeigt
  channel     : enum(email | sms | whatsapp | voice)
  scope       : enum(agency | school)  // agency = global für alle Schulen
  agency_id   : uuid
  school_id   : uuid | null
  subject     : string | null          // nur email
  format      : enum(html | text)       // email = html, sonst text
  body        : html | text             // email = HTML; voice = Sprechtext (TTS); enthält Platzhalter
  attachments : File[]                   // nur email/whatsapp: { name, size, url/storage_key, mime }
  archived    : bool
}
```

**Anhänge:** PDF/Dokumente (z. B. Anfahrt, Stundenplan, AGB) — nur bei **E-Mail** und **WhatsApp** (SMS/Anruf nicht). Im Prototyp werden Name/Größe erfasst; echt: Upload in Storage, `url`/`storage_key`, Größen-/MIME-Limit (z. B. ≤10 MB, PDF/DOC/Bild).

**Editor (siehe Prototyp):** E-Mail wird mit einem **WYSIWYG-Rich-Text-Editor** bearbeitet (Fett/Kursiv, Überschrift, Liste, Link, Button) mit Umschalter **„Visuell ⇄ HTML-Code"** für Profis; Live-Vorschau rendert das HTML mit Beispieldaten. SMS/WhatsApp/Anruf sind Plain Text. Für den echten Versand gilt E-Mail-HTML-Praxis: Inline-Styles, tabellenbasiertes Layout, eingeschränktes CSS (Outlook/Gmail) → Inhalt beim Senden in einen geprüften, responsiven HTML-Wrapper kapseln (z. B. MJML-Compile oder ein festes Template-Gerüst).

- **Platzhalter (Merge-Felder)** im `subject`/`body`, beim Versand ersetzt:
  `{{teilnehmer_name}}`, `{{kurs_titel}}`, `{{termin_datum}}`, `{{termin_uhrzeit}}`, `{{ort}}`, `{{mitbringen}}`, `{{schule_name}}`, `{{storno_link}}`.
- Auswahl im Reminder: `ReminderRule.templates[channel] = template_id`. Nur Vorlagen des passenden Kanals sind wählbar; UI zeigt globale + lokale gemeinsam.
- **Umgesetzt im Prototyp:** Absender-Identität je Schule (§5.1a) und Einwilligung pro Kanal im Buchungsformular (§5.1b).
- **Noch offen fürs Senden:** **Test-Versand**, **Opt-out/Abmelde-Footer** (E-Mail, rechtlich), **Mehrsprachigkeit** (mehrere Sprach-Varianten pro Vorlage), **echte Domain-Verifizierung** (SPF/DKIM/DMARC) statt Badge.

---

## 6. Branding / Look (Theme)

Pro Schule einstellbarer Look (im Prototyp: `applyTheme()` setzt CSS-Variablen live).

```
Branding {
  id, school_id,
  accent_color : string (hex)   // Basis-Akzent; Abstufungen (strong/2/bg/border) werden daraus abgeleitet
  logo_text    : string (max 3) // Kürzel, falls kein Bild
  logo_image   : url | null
  // abgeleitet zur Laufzeit: accent_strong, accent_2, accent_bg, accent_border, hero_grad
}
```

- Default-Akzent: `#dc2626` (Rot).
- Ableitungslogik (dunkler/heller/Tint) ist im Prototyp in `applyTheme()` dokumentiert und sollte serverseitig oder im Frontend identisch nachgebildet werden.

---

## 7. Offene Punkte / Entscheidungen für die Umsetzung

1. **Buchung in einer Serie:** Annahme = Interessent bucht **eine** Session (eine Schnupperstunde), nicht die ganze Serie. (Probe-Block bleibt der Fall „Paket aus mehreren Terminen als eine Buchung".) → bestätigen.
2. **Materialisierungs-Fenster** der Serie: einmalig komplett (für „feste Serie" ausreichend) vs. rollierend. → Default: komplett.
3. ~~Reminder-Opt-in pro Kanal im Anmeldeformular~~ → **erledigt** (§5.1b); ebenso Absender-Identität je Schule (§5.1a).
4. **Audit-Log** für Agentur-Impersonation und Rollenänderungen.
5. **Trainer-Rolle** (Phase 2) später in die Rechte-Matrix aufnehmen.
6. **Detach-Semantik** bei Serien-Änderung final festlegen („diesen Termin" vs. „diesen und folgende" vs. „alle").

---

## Bezug zum Prototyp

| Modell | Prototyp-Stelle (`prototype/index.html`) |
|---|---|
| RecurrenceRule | Serien-Builder S6 + `generateSeries()` / `nthWeekday()` |
| ReminderPolicy | Agentur S11, Schule (Einstellungen), Kurs S6 + `REM`/`renderReminders()` |
| MessageTemplate | Agentur S12 „Vorlagen" (global) + Schule S7d „Vorlagen" (eigene + Kopieren) + Modal `openTemplate()`/`TEMPLATES` |
| Tag (global/lokal) | Agentur S10 „Tags", Schule (Einstellungen), Kurs S6 + `MERKMALE` |
| Membership/Rollen | Agentur S9 (Einladen) + Schule S7c „Team" + `addInviteRow()` |
| SenderIdentity (Agentur + Schule) | Agentur S13 „Versand" + Schule S7b „Einstellungen → Versand" + `SENDER`/`SENDER_AGENCY`/`effSender()` |
| EmailDelivery + use_agency | Agentur S13 (`setSendModeAgency`) + Schule S7b (`toggleSchoolUseAgency`/`setSendMode`) |
| Booking.consent | Anmeldeformular S3 (Opt-in pro Kanal) |
| Course.hero_image (optional) | Kurs-Editor S6 „Kursbild" → Hero-Hintergrund auf S2; ohne Bild Akzent-Verlauf (Fallback) |
| Branding | Schule S7b „Einstellungen / Look" + `applyTheme()` |
