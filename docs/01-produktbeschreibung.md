# Book2Dance — Produktbeschreibung & User Stories

> Arbeitsstand: Entwurf v0.2 · Stand: 09.06.2026
> Zweck dieses Dokuments: Grundlage zur Abnahme mit dem Kollegen. Es beschreibt **was** die App können soll (Scope, Rollen, User Stories), noch nicht **wie** sie technisch gebaut wird. Designflows: `docs/02-designflows.md`, Datenmodell & Rechte: `docs/03-datenmodell.md`.
>
> **Update v0.2:** Der klickbare Prototyp (`prototype/index.html`) ist deutlich über den ursprünglichen MVP hinausgewachsen. Viele ehemalige „Phase 2"-Punkte sind jetzt als Prototyp gebaut — markiert mit **✅ (im Prototyp gebaut)**. Das ist UI-Stand, noch keine fertige Implementierung.

---

## 1. Vision in einem Satz

Eine **Enterprise-Buchungsplattform für Tanzschul-Beratungen**: Eine Agentur (Tanzschulberatung) legt viele Tanzschulen an und verwaltet für jede deren öffentliche **Buchungsseiten für Schnupperkurse** — Interessenten tragen sich so einfach ein wie bei Calendly, die Tanzschule sieht alle Anmeldungen zentral.

---

## 2. Kernkonzept & Abgrenzung

| Referenz | Was wir übernehmen | Was anders ist |
|---|---|---|
| **Calendly** | Reibungsloser Buchungs-Flow für den Endkunden, Bestätigungs-/Erinnerungs-Mails, eingebettete Buchungs-Widgets | Calendly bucht *freie Zeitslots* einer Person. Wir buchen **feste Kurstermine mit begrenzter Platzzahl**. |
| **cal.com** (Open Source) | „Event Types" als Vorlage, Team-/Multi-User-Konzept, Self-Hosting-Gedanke, API-First | Bei uns ist die oberste Ebene eine **Agentur über vielen Tanzschulen** (Mandantenfähigkeit), nicht ein einzelner Nutzer. |
| **Klassische Kursbuchung** (kursifant, SuperSaaS) | Kurse mit Terminen, Kapazität, Warteliste | Wir fokussieren zuerst auf **Schnupperkurse als Lead-Funnel**, nicht auf komplette Kursverwaltung/Abrechnung. |

**Die zentrale Einheit ist der Kurs(-Termin) mit Plätzen, nicht der freie Kalender-Slot.**

---

## 3. Rollen / Personas

1. **Plattform-/Agentur-Admin** (die Tanzschulberatung)
   Legt Tanzschulen (Mandanten) an, verwaltet Zugänge, sieht alles über alle Schulen hinweg. Pflegt globale **Tags**, **Reminder-Standards** und **Vorlagen**. *Das ist der zahlende Enterprise-Kunde.*

2. **Schul-Team mit Rollen** (Mandant) ✅
   Pro Schule mehrere Nutzer mit Rolle — **Superadmin** (volle Kontrolle inkl. Team, Branding, Einstellungen), **Admin** (Kurse & Anmeldungen), **Reader** (nur ansehen). Verwalten Kurse/Schnupperkurse, Termine, Buchungsseite, Anmeldungen. (Rechte-Matrix: `docs/03-datenmodell.md` §2.)

3. **Kursleiter / Trainer** (optional, später)
   Sieht nur eigene Kurse und Teilnehmerlisten. Noch nicht gebaut — die Rolle `reader` deckt „nur lesen" vorerst ab.

4. **Interessent / Endkunde** (anonym, kein Login nötig)
   Findet eine Buchungsseite, sieht verfügbare Schnupperkurse (inkl. kommender Monate), trägt sich in einen Termin ein, wählt **Erinnerungs-Kanäle (Opt-in)**, bekommt Bestätigung.

---

## 4. User Stories

> Format: *Als <Rolle> möchte ich <Ziel>, damit <Nutzen>.* — MVP-relevante Stories sind mit ⭐ markiert.

### Epic A — Agentur verwaltet Tanzschulen (Mandantenfähigkeit)
- ⭐ A1. Als **Agentur-Admin** möchte ich eine neue Tanzschule anlegen (Name, Logo, Standort, Kontakt), damit sie eine eigene Buchungspräsenz bekommt.
- ⭐ A2. ✅ Als **Agentur-Admin** möchte ich pro Tanzschule **mehrere** Nutzer mit **Rollen** (Superadmin/Admin/Reader) einladen, damit die Schule sich selbst verwalten kann.
- A3. Als **Agentur-Admin** möchte ich über alle Schulen hinweg eine Übersicht (Anzahl Buchungen, aktive Kurse) sehen, damit ich den Erfolg pro Mandant beurteilen kann. *(Übersicht im Prototyp, Kennzahlen noch Beispieldaten.)*
- A4. Als **Agentur-Admin** möchte ich eine Tanzschule deaktivieren/archivieren können, ohne Daten zu verlieren.
- A5. ✅ Als **Agentur-Admin/Schule** möchte ich Branding (Farben, Logo) pro Schule setzen, damit Buchungsseiten zur jeweiligen Marke passen. *(Im Prototyp stellt die Schule den Look selbst ein.)*
- A6. ✅ Als **Agentur-Admin** möchte ich **globale Tags, Reminder-Standards und Nachrichten-Vorlagen** einmal zentral pflegen, damit Schulen sie nutzen, ohne alles selbst anzulegen.

### Epic B — Kurse & Schnupperkurse anlegen
- ⭐ B1. Als **Tanzschul-Admin** möchte ich einen Schnupperkurs anlegen (Titel, Beschreibung, Tanzstil, Level, Dauer), damit Interessenten wissen, worum es geht.
- ⭐ B2. ✅ Als **Tanzschul-Admin** möchte ich pro Kurs zwischen **Einzeltermin**, **Serie** (wiederkehrend) und **Probe-Block** (Paket) wählen und konkrete Termine (Datum, Uhrzeit, Ort/Raum) anlegen, auch für kommende Monate.
- ⭐ B3. ✅ Als **Tanzschul-Admin** möchte ich pro Termin eine **maximale Teilnehmerzahl** festlegen, damit der Kurs nicht überbucht wird.
- B4. ✅ Als **Tanzschul-Admin** möchte ich wiederkehrende Termine wie im Kalender (Intervall, Wochentage, monatlich, Ende nach Anzahl/Datum) in **einem** Schritt erzeugen, damit ich nicht jeden einzeln eintrage.
- B5. Als **Tanzschul-Admin** möchte ich einen Kurs als „Paar-Kurs" markieren (Anmeldung zu zweit / mit Partner), damit die Platzlogik stimmt. *(„Kein Partner nötig" o. ä. ist als Tag schon abbildbar; echte Platzlogik offen.)*
- B8. ✅ Als **Tanzschul-Admin** möchte ich Kursen und einzelnen Terminen **Tags** (Chips) geben (global von Agentur + eigene), die auf der Buchungsseite erscheinen.
- B6. Als **Tanzschul-Admin** möchte ich einen Termin absagen/verschieben und automatisch alle Angemeldeten informieren.
- B7. Als **Tanzschul-Admin** möchte ich einen Kurs duplizieren, um ähnliche Angebote schnell zu erstellen.

### Epic C — Buchungsseite (öffentlich)
- ⭐ C1. Als **Interessent** möchte ich auf einer Buchungsseite **alle verfügbaren Schnupperkurse** einer Schule sehen, inkl. der Termine der nächsten Monate.
- ⭐ C2. Als **Interessent** möchte ich Kurse nach Tanzstil/Level/Datum filtern, damit ich schnell das Passende finde.
- ⭐ C3. Als **Interessent** möchte ich pro Kurs eine **kleine Landingpage** sehen (Beschreibung, was mitbringen, Ort, freie Plätze), damit ich mich sicher anmelde.
- ⭐ C4. ✅ Als **Interessent** möchte ich mich mit minimalen Daten (Name, E-Mail, Telefon) für einen Termin eintragen — ohne Account — und dabei **pro Kanal wählen**, wie ich erinnert werde (E-Mail/SMS/WhatsApp/Anruf, Opt-in).
- ⭐ C5. Als **Interessent** möchte ich nach der Anmeldung eine **Bestätigungs-Mail** mit allen Details (und Kalender-Eintrag) bekommen.
- C6. Als **Interessent** möchte ich mich auf eine **Warteliste** setzen, wenn ein Termin voll ist, und nachrücken können.
- C7. Als **Interessent** möchte ich meine Anmeldung über einen Link **stornieren/umbuchen** können.
- C8. Als **Interessent** möchte ich die Buchungsseite eingebettet auf der Webseite der Tanzschule nutzen können (Embed/iFrame).

### Epic D — Anmeldungen verwalten
- ⭐ D1. Als **Tanzschul-Admin** möchte ich pro Termin die **Teilnehmerliste** sehen (Name, Kontakt, Status), damit ich den Kurs vorbereiten kann.
- ⭐ D2. Als **Tanzschul-Admin** möchte ich eine neue Anmeldung sofort als Benachrichtigung erhalten, damit ich Leads schnell nachfassen kann.
- D3. ✅ Als **Tanzschul-Admin** möchte ich Teilnehmer als „angemeldet → erschienen → konvertiert" markieren, um den Funnel-Erfolg zu messen.
- D4. ✅ Als **Tanzschul-Admin** möchte ich Teilnehmerlisten exportieren (CSV) oder in mein Tool übergeben.
- D5. ✅ Als **Tanzschul-Admin** möchte ich manuell jemanden eintragen (Telefon-Anmeldung), damit alle Anmeldungen an einem Ort sind.

### Epic E — Benachrichtigungen, Erinnerungen & Vorlagen
- ⭐ E1. Als **System** möchte ich automatische Bestätigungs-Mails versenden.
- E2. ✅ Als **System** möchte ich vor dem Termin **Reminder** senden (E-Mail/SMS/WhatsApp/Anruf), um No-Shows zu reduzieren. Regeln **dreistufig vererbt** (Agentur → Schule → Kurs), Zeitpunkte frei konfigurierbar.
- E3. ✅ Als **Agentur/Schule** möchte ich **Nachrichten-Vorlagen** pro Kanal anlegen/bearbeiten — **E-Mail als HTML** (WYSIWYG + Code-Modus), SMS/WhatsApp/Anruf als Text — mit **Platzhaltern** und Vorschau; global (Agentur) + eigene (Schule, inkl. Kopieren).
- E5. ✅ Als **Schule** möchte ich meine **Absender-Identität** setzen (From-Name, Reply-To, verifizierte Domain, SMS-Sender-ID) und den **Versandweg** wählen (zentral über Book2Dance mit DKIM **oder** eigener SMTP/ESP).
- E6. ✅ Als **Interessent** möchte ich pro Kanal **einwilligen** (Opt-in), wie ich erinnert werde; jederzeit widerrufbar.
- E4. Als **System** möchte ich nach dem Schnupperkurs eine Follow-up-Mail („Lust auf den vollen Kurs?") senden. *(noch offen)*
- E7. Als **Schule** möchte ich eine **Test-Mail/-SMS** an mich senden, bevor sie live geht. *(noch offen)*

### Epic F — Konto, Rechte, Einstellungen
- ⭐ F1. Als **Nutzer** möchte ich mich sicher einloggen. Rollen: Agentur-Admin sowie pro Schule **Superadmin/Admin/Reader** ✅ *(Rollen & Team-Verwaltung im Prototyp; echtes Login/Auth offen)*.
- F2. Als **Schul-Admin** möchte ich Öffnungszeiten/Standorte/Räume pflegen.
- F3. Als **Trainer** möchte ich nur meine eigenen Kurse und Teilnehmer sehen. *(Trainer-Rolle noch offen; `reader` deckt „nur lesen" vorerst ab.)*

---

## 5. Feature-Scope: Stand Prototyp vs. offen

**✅ Im Prototyp gebaut (UI-Stand):**
Mandanten anlegen + **Mehrfach-Team mit Rollen** (A1, A2) · Schnupperkurs + **Einzeltermin/Serie/Probe-Block** + Kapazität (B1–B4) · **Tags** an Kurs/Termin, global + lokal (A6, B8) · Buchungsseite mit Übersicht + Filter (C1–C3) · Anmeldung ohne Account + **Reminder-Opt-in** (C4) · Teilnehmerliste + Lead-Benachrichtigung + **Funnel-Status** + CSV + manuell (D1–D5) · **Reminder dreistufig** über E-Mail/SMS/WhatsApp/Anruf (E2) · **Nachrichten-Vorlagen** (HTML-E-Mail + Text) mit Platzhaltern, global + lokal (E3) · **Absender-Identität + Versandweg** (managed/eigener SMTP) (E5, E6) · **Branding/Look pro Schule** (A5).

> „Gebaut" = klickbarer Prototyp mit Beispieldaten, noch keine Persistenz/Backend/echtes Auth.

**Noch offen:**
Login/Auth & echte Rechte-Durchsetzung (F1) · Schule archivieren (A4) · echte Agentur-Kennzahlen (A3) · Paar-/Platzlogik (B5) · Warteliste (C6) · Stornierung/Umbuchung-Flow (C7) · Embed/iFrame (C8) · Follow-up-Mail (E4) · Test-Versand (E7) · Trainer-Rolle (F3) · Räume/Öffnungszeiten (F2).

**Phase 3 / Vision:** Bezahlung für kostenpflichtige Kurse, vollständige Kursverwaltung, Analytics-Dashboard, Mobile App.

---

## 6. Screens (Basis für die Designflows)

**Öffentlich (Endkunde):**
1. Buchungsseite der Schule — Kursliste + Filter
2. Kurs-Landingpage — Detail + Terminauswahl + freie Plätze
3. Anmelde-Formular (Calendly-Style)
4. Bestätigungsseite + Bestätigungsmail

**Backend Tanzschule:**
5. Dashboard (kommende Termine, neue Anmeldungen)
6. Kurs anlegen/bearbeiten + **Serie/Wiederholung** + Termine + Tags + Reminder
7. Teilnehmerliste pro Termin (Funnel-Status, CSV)
7c. **Team / Rollen** (mehrere Admins einladen)
7d. **Vorlagen** (eigene + globale, Kopieren)
8. Einstellungen: **Look/Branding, Tags, Reminder-Standard, Absender, E-Mail-Versand**

**Backend Agentur:**
9. Übersicht aller Tanzschulen
10. Tanzschule anlegen / **mehrere** Nutzer mit Rollen einladen
11. **Tags** (Kurs-Tags-Katalog, global)
12. **Reminder-Standards** (global)
13. **Vorlagen** (global)

---

## 7. Entscheidungen & offene Punkte

**Entschieden (08.06.2026):**
1. **Bezahlung:** MVP ist **kostenlos** — reine Anmeldung. Stripe-Bezahlung erst Phase 3.
2. **Schnupperkurs-Modell:** **Beides möglich** — die Schule wählt pro Kurs zwischen *Einzeltermin (Drop-in)* und *mehrwöchigem Probe-Block*. (Datenmodell muss beide Fälle abbilden, siehe B2.)
3. **Design-Stil:** **shadcn** als Basis (moderner SaaS-Look, trägt Backend wie Buchungsseite).

**Entschieden (09.06.2026, im Prototyp umgesetzt):**
8. **Rollen pro Schule:** Superadmin / Admin / Reader (mehrere Admins pro Schule). Trainer-Rolle später.
9. **Wiederholung:** **feste Serie** (kalender-artig: Intervall, Wochentage, monatlich; Ende nach Anzahl/Datum). Jeder Termin bleibt einzeln editier-/absagbar.
10. **„Tags"** statt „Merkmale"; heutiger Katalog = Kurs-Tags. Global (Agentur) + lokal (Schule).
11. **Reminder dreistufig** (Agentur → Schule → Kurs), Kanäle E-Mail/SMS/WhatsApp/Anruf, Zeitpunkte frei.
12. **Vorlagen** pro Kanal, **E-Mail als HTML**; global + lokal; Auswahl pro Reminder-Kanal.
13. **Versand:** zentral über Book2Dance (per-Schule **DKIM**), optional **eigener SMTP/ESP** pro Schule. **Opt-in pro Kanal** im Anmeldeformular (DSGVO).

→ Datenmodell & Rechte-Matrix dazu: `docs/03-datenmodell.md`.

**Noch offen (mit Kollege zu klären):**
4. **Paar-Anmeldung** („mit Partner anmelden") — Annahme: später; „kein Partner nötig" als Tag bereits abbildbar.
5. **Mehrsprachigkeit** — Annahme: nur Deutsch im MVP (Vorlagen später mehrsprachig).
6. **Account für Endkunden** — bewusst ohne Login (wie Calendly). Annahme: bestätigt.
7. **Eigene echte Landingpage** pro Schule zusätzlich zur Buchungsseite — aktuell nicht im MVP.
14. **Persistenz/Backend & echtes Login/Auth** — der Prototyp hält noch keine Daten; nächster großer Schritt.
15. **Test-Versand, Abmelde-Footer/STOP, echte Domain-Verifizierung** — für den Live-Versand nötig.

---

## Referenzen
- Calendly Features — https://calendly.com/features
- cal.com (Open Source) — https://cal.com/
- Kursifant (Tanzschul-Kursbuchung) — https://kursifant.com
- SuperSaaS (Terminbuchung) — https://www.supersaas.de/info/tanzstunden_online_terminbuchungssystem
