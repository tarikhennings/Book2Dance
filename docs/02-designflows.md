# Book2Dance — Design-Flows

> Arbeitsstand: Entwurf v0.2 · Stand: 09.06.2026
> Zweck: Abnahme der User-Journeys & Screen-Struktur. Bezug: `docs/01-produktbeschreibung.md` (Scope), `docs/03-datenmodell.md` (Modell/Rechte).
> Design-Basis: **shadcn** (moderner SaaS-Look). Default-Akzentfarbe **Rot**, pro Schule änderbar.
> Legende: `[Button]` · `{Eingabefeld}` · `→` Navigation · `⟳` System-Aktion (z.B. Mailversand) · **✅** = im klickbaren Prototyp gebaut (`prototype/index.html`).
>
> **Update v0.2:** Über den ursprünglichen MVP hinaus sind im Prototyp dazugekommen: **Serien-Termine, Tags, Team/Rollen, Reminder (mehrkanalig, dreistufig), Nachrichten-Vorlagen, Absender/Versand**. Screens unten entsprechend erweitert.

---

## Überblick: 3 Welten

```
ÖFFENTLICH (Endkunde, kein Login)   BACKEND SCHULE (Login)                       BACKEND AGENTUR (Login)
Buchungsseite → Kurs → Anmeldung    Dashboard · Kurse · Teilnehmer · Team ·      Schulen · Schule anlegen ·
   → Bestätigung                    Vorlagen · Einstellungen                     Tags · Reminder · Vorlagen
```

---

## Flow 1 — Endkunde bucht einen Schnupperkurs (Herzstück)

```
(1) Buchungsseite der Schule
        │  Kunde filtert/scrollt, sieht alle Schnupperkurse inkl. kommender Monate
        ▼
(2) Kurs-Landingpage
        │  liest Details, wählt einen Termin (Einzeltermin) ODER den Block
        ▼  [Diesen Termin buchen]
(3) Anmeldeformular  (Calendly-Style, ein Schritt)  ✅
        │  {Name} {E-Mail} {Telefon}  · sieht gewählten Termin + freie Plätze
        │  Reminder-Opt-in: ☑ E-Mail  ☐ SMS  ☐ WhatsApp  ☐ Anruf  (SMS/WA/Anruf brauchen Telefon)
        │  ☑ Datenschutz
        ▼  [Verbindlich anmelden]
   ⟳ System: Platz reservieren · Bestätigungsmail + Kalender (.ics) senden · Schule benachrichtigen · Reminder gem. Opt-in einplanen
        ▼
(4) Bestätigungsseite  „Du bist dabei!" + Termin-Details + Reminder-Kanäle + [Zum Kalender] [Stornieren-Link in Mail]  ✅
```

**Sonderfälle:**
- Termin **voll** → Termin ausgegraut „ausgebucht" ✅ ([Auf Warteliste] noch offen).
- Kunde schließt vor Absenden → keine Reservierung.
- Doppelte Anmeldung (gleiche Mail, gleicher Termin) → freundlicher Hinweis statt Fehler.

---

## Flow 2 — Tanzschule legt einen Schnupperkurs an

```
(1) Login → Dashboard
        ▼ [Neuer Kurs]
(2) Kurs-Grunddaten  ✅
        │  {Titel} {Tanzstil} {Level} {Beschreibung} {Was mitbringen} {Ort/Raum}
        │  Tags: Chips aus Katalog (global + eigene) + freier Tag                ✅
        │  Modell wählen:  ( ) Einzeltermin   (•) Serie   ( ) Probe-Block         ✅
        ▼
(3) Termine festlegen  ✅
        │  Serie:        {Start} {Uhrzeit} {Dauer} · Wiederholen: alle {N} Tag/Woche/Monat
        │                · Wochentage (Mo–So) · monatlich am Datum / am n-ten Wochentag
        │                · Ende: nach {Anzahl} ODER bis {Datum}  → [Termine erzeugen]
        │  Einzeltermin: eine Zeile · Block: mehrere Zeilen als Paket
        │  Erzeugte Termine als Table (Datum/Uhrzeit/Plätze/Tags) — einzeln editier-/löschbar  ✅
        ▼
(4) Reminder für den Kurs  ✅
        │  ☑ Schul-Standard verwenden  — oder eigene Regeln pro Kurs
        │  je Regel: Zeitpunkt vorher + Kanäle (E-Mail/SMS/WA/Anruf) + Vorlage je Kanal
        ▼ [Veröffentlichen]
   ⟳ Kurs + alle Termine erscheinen sofort auf der Buchungsseite
        ▼
(5) Kurs-Detail im Backend  → Status, Termine, Anmeldungen pro Termin
```

---

## Flow 3 — Tanzschule sieht & verwaltet Anmeldungen

```
Dashboard zeigt „3 neue Anmeldungen"
        ▼
Teilnehmerliste (pro Termin):  Name · Kontakt · Status (angemeldet → erschienen → konvertiert)  ✅
        ├─ [Manuell hinzufügen]  (Telefon-Anmeldung)  ✅
        ├─ [Als erschienen / konvertiert markieren]   ✅  (Funnel-Messung)
        └─ [CSV exportieren]   ✅
```

---

## Flow 4 — Agentur legt eine neue Tanzschule an

```
Agentur-Login → Übersicht aller Schulen (Kacheln: Name, #Kurse, #Buchungen)  ✅
        ▼ [Schule anlegen]
{Name} {Logo/Kürzel} {Standort} {Buchungsseiten-URL}                          ✅
        ▼
Admins einladen:  mehrere Zeilen {Name} {E-Mail} {Rolle: Superadmin/Admin/Reader}  ✅
        ▼ [Anlegen & einladen]   ⟳ Einladungsmails
        ▼
Schule erscheint in Übersicht · eigene Buchungsseiten-URL wird erzeugt
```

> Globale Vorgaben der Agentur (für alle Schulen): **Tags-Katalog**, **Reminder-Standards**, **Vorlagen** — eigene Screens, von jeder Schule nutz-/überschreibbar.

---

## Flow 5 — Reminder & Vorlagen einrichten (dreistufig)  ✅

```
Agentur setzt Standards          Schule übernimmt oder überschreibt        Kurs überschreibt optional
Reminder-Standards (global)  →   ☑ Agentur-Standard verwenden  /  eigene   →  ☑ Schul-Standard / eigene
Vorlagen (global)            →   eigene Vorlagen + [Kopieren] globaler
        │
        ▼ Vorlage anlegen (Modal)
  {Name} {Kanal: E-Mail/SMS/WhatsApp/Anruf}
  E-Mail: Betreff + HTML-Editor (visuell ⇄ Code) · sonst Textfeld
  Platzhalter einfügen ({{teilnehmer_name}} …) · Live-Vorschau (Von-Zeile aus Absender)
        ▼ je Reminder-Regel: Kanal aktivieren → Vorlage je Kanal wählen (mit Namen sichtbar)
```

---

## Flow 6 — Schul-Team & Look einrichten  ✅

```
Schule → Team:        Mitglieder-Tabelle (Name/E-Mail/Rolle/Status) · [Mitglied einladen] mit Rolle
Schule → Einstellungen:
   Look:              Akzentfarbe (Presets + frei) · Logo-Kürzel/-Bild  → Live-Vorschau, global wirksam
   Tags:              globale (read-only) + eigene Tags
   Reminder-Standard: ☑ Agentur-Standard / eigene Regeln
   Absender:          From-Name · Reply-To · Absende-Adresse (✓ verifiziert) · SMS-Sender-ID · WhatsApp
   E-Mail-Versand:    (•) Über Book2Dance (DKIM)   ( ) Eigener SMTP {Host/Port/User/Pass/Verschlüsselung}
```

---

## Screen-Wireframes (Aufbau, shadcn-Komponenten)

### S1 — Buchungsseite der Schule *(öffentlich)* ✅
- **Header:** Schul-Logo + Name, kurzer Claim.
- **Filterleiste:** `Select` Tanzstil · `Select` Level · Zeitraum · `Input` Suche. **Mobil:** kompakte, horizontal scrollbare Pills (eine Zeile) statt gestapelter Felder.
- **Kursliste:** Karten **ohne Bild** (kompakt, v. a. mobil wichtig) — Bild gibt's auf der Detailseite (S2).
- **Kursliste:** `Card`-Grid. Pro Card: Titel, Tanzstil-`Badge`, nächster Termin, „X Plätze frei", `[Ansehen]`.
- **Monats-Gruppierung:** Tabs oder Section-Header je Monat (auch kommende Monate sichtbar).
- *Leerzustand:* „Aktuell keine Schnupperkurse — schau bald wieder vorbei."

### S2 — Kurs-Landingpage *(öffentlich)* ✅
- **Hero mit Kursbild** (optional): Hintergrundfoto hinter Chips + Titel, dunkler Verlauf für Lesbarkeit, weiße Schrift. **Ohne Bild** kompakter Akzent-Verlauf (Fallback).
- **Hero:** Titel, Tanzstil/Level-`Badge`s, Kurzbeschreibung.
- **2-Spalten:** links Details (Beschreibung, Was mitbringen, Ort + Karte); rechts **Termin-Auswahl** (`RadioGroup` der Termine, jeweils Datum/Uhrzeit + „X frei"), darunter sticky `[Diesen Termin buchen]`.
- Bei Block: ein Paket-Block statt Einzelterminauswahl.

### S3 — Anmeldeformular *(öffentlich, Modal oder eigene Seite)* ✅
- Zusammenfassung des gewählten Termins oben (read-only).
- Felder: `{Name}` `{E-Mail}` `{Telefon}` `{Nachricht}`.
- **Reminder-Opt-in:** `Checkbox`-Block (E-Mail vorausgewählt; SMS/WhatsApp/Anruf mit Hinweis „Telefon nötig").
- `Checkbox` Datenschutz · `[Verbindlich anmelden]` (primär). Inline-Validierung.

### S4 — Bestätigung *(öffentlich)*
- Großer Erfolgs-`Alert`: „Du bist angemeldet!"
- Termin-Details, `[Zum Kalender hinzufügen]`, Hinweis auf Bestätigungsmail.

### S5 — Schul-Dashboard *(Login)* ✅
- `Sidebar` (Dashboard · Kurse · Teilnehmer · **Team · Vorlagen** · Einstellungen).
- Oben Kennzahlen-`Card`s: kommende Termine, neue Anmeldungen, Auslastung.
- Liste „Nächste Termine" + „Neueste Anmeldungen".

### S6 — Kurs anlegen/bearbeiten *(Login)* ✅
- Linke Spalte: Grunddaten → **Tags** → **Kurs-Modell (Einzeltermin/Serie/Probe-Block)** → **Serien-Builder** → erzeugte **Termine** (`Table`, Tags/Plätze pro Termin) → **Reminder** (erben oder eigene, Vorlage je Kanal).
- Rechte Spalte: sticky **Live-Vorschau** der Buchungs-Card.

### S7 — Teilnehmerliste *(Login)* ✅
- `Table`: Name · Kontakt · Angemeldet · Status-`Badge` · Aktion (→ erschienen → konvertiert).
- Funnel-Zähler-`Badge`s · Toolbar: `[Manuell hinzufügen]` · `[CSV-Export]`.

### S7c — Team / Rollen *(Login)* ✅
- Mitglieder-`Table`: Name · E-Mail · **Rolle (`Select`: Superadmin/Admin/Reader)** · Status · Entfernen.
- „Mitglied einladen": Zeilen {Name}{E-Mail}{Rolle} + Rollen-Legende.

### S7d — Vorlagen (Schule) *(Login)* ✅
- „Eigene Vorlagen" (`[Bearbeiten]`/`[✕]`) + „Globale Vorlagen" (read-only, `[Ansehen]`/`[Kopieren]`).
- Editor als **Modal** (s. Flow 5). Platzhalter-Panel.

### S8 — Einstellungen (Schule) *(Login)* ✅
- Karten: **Look/Branding** (Farbe, Logo, Live-Vorschau) · **Tags** · **Reminder-Standard** · **Vorlagen** (Verweis) · **Absender** · **E-Mail-Versand** (managed/eigener SMTP).

### S9 — Agentur-Übersicht *(Login)* ✅
- `Card`-Grid aller Schulen + `[Schule anlegen]`. Pro Schule: #Kurse, #Buchungen, „Verwalten".
- Sidebar: Tanzschulen · Auswertungen · **Tags · Reminder · Vorlagen** · Nutzer · Einstellungen.

### S10–S13 — Agentur global *(Login)* ✅
- **Tags** (Kurs-Tags-Katalog) · **Reminder-Standards** (dreistufig vererbt) · **Vorlagen** (global) · Schule anlegen + Admins mit Rollen einladen.

---

## Was als Nächstes
1. ✅ Klickbarer **HTML-Prototyp** (`prototype/index.html`) — deckt alle Flows oben mit Beispieldaten ab.
2. Kollege nimmt Flows + Screens am Prototyp ab / kommentiert.
3. **Tech-Stack & echtes Backend**: Persistenz, Login/Auth + Rechte-Durchsetzung (Rollen), Versand-Anbindung (ESP + DKIM, SMS/WhatsApp/Voice-Provider).
4. Offene Flows ergänzen: Warteliste, Stornierung/Umbuchung, Embed, Follow-up-Mail, Test-Versand.
