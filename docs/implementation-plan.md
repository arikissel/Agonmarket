# Agonmarket — Implementierungsplan v1

> Basiert auf der freigegebenen [Spec](./spec.md) vom 12.08.2026.
> **v1-Ziel:** Eine vollständige Runde, gespielt von 8–10 echten Freunden über Expo Go.
> **Kontext:** Solo, nebenberuflich. Sprints sind als „Abende + ein Wochenende" gedacht,
> nicht als Kalenderwochen mit Vollzeit.

---

## Grundsätze für den ganzen Bau

1. **Die Punktelogik lebt ausschließlich in Postgres.** Eine einzige Quelle der Wahrheit,
   direkt neben den Daten, transaktional und idempotent. Der Client rechnet nie Punkte aus —
   er zeigt nur an, was der Server verbucht hat.
2. **Sicherheit über RLS, nie über den Client.** Verdeckte Tipps, Gruppenzugehörigkeit und
   Admin-Rechte werden in Policies erzwungen. Ein manipulierter Client darf nichts sehen,
   was er nicht darf.
3. **Jede Migration ist eine Datei**, versioniert unter `supabase/migrations/`. Keine
   Klick-Änderungen im Studio, die nirgends dokumentiert sind.
4. **Keine rohen Farbwerte in Komponenten.** Ausschließlich Tokens — das ist die Voraussetzung
   dafür, dass Light Mode kein Refactoring wird.
5. **Wording-Disziplin ab dem ersten String:** tippen, Punkte, Rangliste, Champion.
   Nie wetten, Einsatz, Quote, Gewinn — auch nicht in Platzhaltern oder Kommentaren.

---

## Sprint 0 — Fundament

**Ziel:** Die App startet in Expo Go, hat Navigation, ein Theme-System und redet mit Supabase.

- Abhängigkeiten: `expo-router`, `@supabase/supabase-js`, `@tanstack/react-query`, `zustand`,
  `expo-secure-store`, `expo-notifications`, `expo-linking`, `react-native-url-polyfill`
- `app.json`: `scheme: "agonmarket"`, `userInterfaceStyle: "automatic"` (steht aktuell auf
  `"light"`), expo-router Plugin
- Umstieg von `App.tsx` auf expo-router: `app/_layout.tsx` mit Providern, Tab-Layout
  `app/(tabs)/` für *Home* / *Tippen* / *Profil*
- **Design-Token-System** — das Herzstück für beide Farbmodi:
  - `src/theme/tokens.ts` — `dark` und `light` mit identischen Schlüsseln, Werte exakt nach
    [spec.md § 5](./spec.md#5-architektur): `background`, `surface`, `surfaceRaised`,
    `textPrimary`, `textMuted`, `border`, **`accent`** (Violett `#7C5CFF` / `#6A45E8`),
    `success` (Mint), `award` (Gold), `categorySport`, `categoryFinance`, `categoryPolitics`
  - **Die drei Akzente sind semantisch getrennt** — `accent` für Aktion/Auswahl/„das bist du",
    `success` nur für richtige Tipps und gewonnene Punkte, `award` nur für Rang 1 und Badges.
    Wer einen primären Button in `success` einfärbt, hat den Fehler gemacht, den die Trennung
    verhindern soll
  - `src/theme/ThemeProvider.tsx` — Modus `system | dark | light`, in SecureStore persistiert
  - `src/theme/useStyles.ts` — memoisierter Hook, der eine Style-Factory gegen die Tokens auflöst
  - `src/theme/type.ts` — Schriftgrößen-Skala, inkl. der „großen markanten Zahlen" als eigene Rolle
- Supabase-Projekt in **EU Frankfurt** anlegen, `.env` + `app.config.ts` für die Keys
- TypeScript strict, Pfad-Alias `@/*`

**Verifiziert wenn:** App läuft auf dem eigenen Handy in Expo Go, Tabs schalten um, ein
Theme-Umschalter in *Profil* wechselt sichtbar zwischen Dark und Light, `select 1` gegen
Supabase liefert ein Ergebnis.

---

## Sprint 1 — Auth, Profil, Rechtliches

**Ziel:** Einloggen, Username setzen, dauerhaft eingeloggt bleiben, Account löschen können.

- Migration: `profiles` (`id → auth.users`, `username` unique citext, `avatar_key`,
  `push_token`, `tos_accepted_at`, `age_confirmed_at`, `is_admin`) + RLS
  + Trigger `on auth.users insert → profiles`
- Supabase-Client mit SecureStore-Adapter als Session-Storage (nicht AsyncStorage —
  Refresh-Tokens gehören verschlüsselt)
- Magic-Link-Flow: `app/(auth)/welcome.tsx` → `email.tsx` → `check-inbox.tsx`
  (60 s Cooldown, „E-Mail ändern"), Redirect in v1 über `exp://` (Expo Go),
  `agonmarket://` bereits mitkonfiguriert für den späteren Dev-Build
- `app/(auth)/setup.tsx`: Username mit entprelltem Verfügbarkeits-Check (RPC, nicht
  Tabellen-Select), Avatar-Presets, **AGB/Datenschutz-Checkbox + 18+-Bestätigung**,
  beide Zeitstempel werden gespeichert
- Edge Function `delete-account`: löscht Profil + Auth-User, in *Profil → Einstellungen*
- Fehlerzustände: ungültige E-Mail inline, abgelaufener Link mit Neu-senden, Offline-Banner

**Verifiziert wenn:** Login auf einem echten Handy funktioniert, App-Neustart hält die Session,
ein zweiter Login mit derselben Adresse führt direkt in die App (kein zweiter Setup-Screen),
Account-Löschung entfernt beide Datensätze.

---

## Sprint 2 — Gruppen & Invite

**Ziel:** Zwei Handys landen über einen WhatsApp-Link in derselben Gruppe.

- Migration: `groups` (`invite_code` unique, kurz und verwechslungsarm — kein 0/O/1/I),
  `group_members` (`role`, unique auf `(group_id, profile_id)`) + RLS:
  Mitglieder sehen ihre Gruppen, Beitritt nur über eine `SECURITY DEFINER`-RPC
  `join_group_by_code(code)` — die Tabelle selbst bleibt für Fremde unsichtbar
- Gruppe erstellen (Name + Emoji) → Invite-Sheet öffnet sich sofort, `expo-sharing`
- Beitritt per Code-Eingabe
- **Invite-Kontext über den Mail-Umweg:** Code wird *vor* dem Magic-Link-Versand in
  SecureStore gelegt und nach erfolgreichem Login eingelöst und gelöscht
- Home: Gruppenliste mit Empty State und den zwei Wegen aus Journey B

**Verifiziert wenn:** Handy A erstellt eine Gruppe, schickt den Link per WhatsApp, Handy B
installiert nichts, öffnet den Link, loggt sich ein — und steht in der Gruppe, ohne den Code
je selbst eingetippt zu haben.

---

## Sprint 3 — Katalog, Admin-Web, Runden

**Ziel:** Du kannst Fragen anlegen, ein Host kann daraus eine Runde bauen.

- Migration: `events`, `event_markets` (inkl. `settlement_source`, `settlement_rule`,
  `settlement_at`, `settlement_type`, `status`), `rounds`, `round_events`
  - **Unveränderlichkeit:** Trigger, der `settlement_source` / `settlement_rule` /
    `settlement_at` nach `status = 'published'` gegen Änderung sperrt
  - Katalog global lesbar für alle eingeloggten Nutzer, schreibbar nur für `is_admin`
- **Admin-Web** (`admin/`, Next.js auf Vercel): Supabase-Login mit Admin-Prüfung,
  Event + Märkte anlegen, Liste offener Märkte, Freigabe-Queue mit `status: draft → published`
  (die Queue ist bewusst schon da — später schreibt der KI-Generator in genau diese Tabelle)
- Runden-Erstellung in der App, 4 Schritte:
  1. Name + **Modus** (Täglich/Klassisch, Default Täglich) + Dauer (Default 7 Tage) +
     bei Täglich `questions_per_day` (3/5/7, Default 5) und `unlock_time` (Default 09:00)
  2. Credits (Default 1000); `max_stake_per_question` wird mit 10 % vorbelegt
  3. Fragen (Kategorie-Filter, mind. 1) — inkl. Sektion **„Beliebt diese Woche"** aus
     `trending_markets`, ab 3 aktiven Runden eingeblendet
  4. Übersicht mit optionaler ×2-Gewichtung
- Beim Anlegen wird für jedes `round_event` das `unlock_at` berechnet und festgeschrieben
  (Klassisch = Rundenstart). Der Wizard lehnt Fragen ab, deren Deadline vor ihrem `unlock_at`
  läge
- RLS: `round_events` mit `unlock_at > now()` sind für Mitglieder **nicht lesbar** — sonst zieht
  ein Client die kommenden Tagesfragen vorab aus der API
- Materialized View `trending_markets` + Unique-Index + `pg_cron`-Refresh (15 Min).
  Enthält ausschließlich `(event_market_id, active_round_count)` — kein Gruppenbezug

**Verifiziert wenn:** Ein im Admin-Web angelegter Markt taucht in der Fragenauswahl der App auf
und lässt sich in eine Runde legen; ein Versuch, `settlement_rule` nach Veröffentlichung zu
ändern, wird von der Datenbank abgelehnt.

---

## Sprint 4 — Tippen

**Ziel:** Die Frage-Karte, das zentrale Element der App.

- Migration: `predictions` (`answer jsonb`, `stake`, unique auf
  `(round_id, market_id, profile_id)`) + **die kritische RLS-Policy:**
  eigene Tipps immer lesbar, fremde erst wenn `now() >= settlement_at` oder
  `status in ('locked','settled')`
- CHECK-Constraints in der DB, nicht nur im Client: `stake > 0` und
  `stake <= rounds.max_stake_per_question` (Default 100 = 10 % von 1000). Kein
  Fallback auf `budget / Fragenanzahl` mehr — der war nur nötig, solange Restcredits verfielen
- Serverseitige Prüfung, dass die Summe aller Einsätze eines Spielers `credit_budget` nicht
  überschreitet, und dass `now() >= round_events.unlock_at`
- `FrageKarte` in vier Zuständen (offen / getippt / gesperrt / ausgewertet), beide Farbmodi
- Eingabekomponenten: Chips (Typ A), zwei große Buttons (Typ B), Stepper und Zahlenfeld (Typ C)
- Budget-Leiste im verbindlichen Format **„640 von 1000 Credits · noch 20 Fragen offen"** —
  beide Zahlen plus Restfragen, sonst ist der Wert nicht interpretierbar. Dazu Cap-Hinweis am
  Maximum, Auto-Save mit Häkchen, jederzeit änderbar bis zur Deadline
- Gruppen-Verteilung erst nach eigenem Tipp, über eine Aggregat-RPC (die reine Verteilung
  verrät niemanden namentlich)
- **App-weite Verteilung** als zweite Leiste („54 % aller Tipper"):
  `market_distribution_cache` + `pg_cron` (5 Min) + RPC `get_market_distribution()` als
  `SECURITY DEFINER` mit `SET search_path = ''`, Eigener-Tipp-Prüfung **in der Funktion**,
  Schwelle < 20 Tipps → keine Ausgabe, Typ C nur Top 3 + „Sonstige" mit Unterdrückung
  unter n = 5. **Kein Service-Role-Key im Client.** UI bleibt in v1 unsichtbar, weil 8–10
  Tester die Schwelle nie erreichen
- *Tippen*-Tab: alle offenen Tipps über alle Gruppen, nach Deadline sortiert

**Verifiziert wenn:** Zwei Accounts tippen; ein direkter SQL-Select als Account A auf die
Tipps von Account B liefert vor der Deadline **null Zeilen** und danach die Daten. Ein Einsatz
über dem Cap wird von der Datenbank abgelehnt, nicht nur vom Formular.

---

## Sprint 5 — Settlement & Leaderboard

**Ziel:** Der Moment, über den in der Gruppe geredet wird. Und der Teil, der stimmen muss.

- Migration: `results`, `scores` (unique auf `(round_id, market_id, profile_id)`)
- **`settle_market(market_id, outcome jsonb)`** — idempotent: zweimal ausgeführt entsteht
  exakt derselbe Zustand. Berechnet Auszahlungen nach den festen Multiplikatoren
  (A/B = Anzahl Optionen · C Spielergebnis = ×4/×2/0 · C Wert = ±1 → ×4, ±3 → ×2, sonst 0),
  multipliziert mit der Gewichtung, schreibt `scores`, setzt `status = 'settled'`
- **`void_market(market_id, grund)`** — annulliert, erstattet alle Einsätze zurück ins Budget,
  nimmt den Markt aus der Wertung. Muss auch einen bereits abgerechneten Markt sauber zurückdrehen
- **Endstand = Summe der Auszahlungen + nicht gesetzte Rest-Credits.** Auf dem Leaderboard
  werden beide Bestandteile getrennt ausgewiesen
- **SQL-Tests** (`supabase/tests/scoring.sql`) — der einzige Teil mit echtem Testzwang:
  jeder Fragetyp, Gewichtung ×2, Cap-Grenzfall, Annullierung nach Abrechnung, doppeltes
  Settlement, Runde mit 1 Frage, Spieler ohne jeden Tipp (landet auf **genau `credit_budget`**,
  also 1000), und der Erwartungswert-Nachweis: ein Zufallstipper mit Trefferquote `1/n` kommt
  im Mittel ebenfalls auf `credit_budget` — die Grundlage von § 2.2 der Spec
- Adapter `supabase/functions/settle-sport` (API-Football) und `settle-finance`
  (CoinGecko/Twelve Data, Stichtag 22:00 Europe/Berlin, Aktien = letzter Handelstag),
  beide gebatcht und gecacht wegen der Free-Tier-Limits
- `pg_cron`: Märkte sperren bei Deadline, danach Settlement-Versuch mit Backoff
- Leaderboard (eigene Zeile hervorgehoben, Auf-/Absteiger-Pfeile) und Auswertungs-Screen
  (wer tippte was, wer holte wie viel)

**Verifiziert wenn:** Eine komplette Testrunde wird durchgerechnet und die Ergebnisse stimmen
von Hand nachgerechnet. `settle_market` zweimal ausgeführt ändert nichts. Ein Spieler, der nie
getippt hat, steht mit exakt 1000 Punkten in der Tabelle — über allen, die schlechter als der
Zufall lagen, und unter allen, die informiert getippt haben.

---

## Sprint 6 — Push, Deep Links, Siegerehrung

**Ziel:** Der Engagement-Loop schließt sich.

- Push-Token nach dem **ersten abgegebenen Tipp** anfragen — eigener Erklär-Screen vor dem
  Systemdialog, nie im Onboarding
- `pg_cron`-Jobs für drei Auslöser, Priorität 1 → 3: (1) T-2h vor der frühesten offenen
  Deadline, **nur an Nicht-Tipper**, pro Runde gebündelt; (2) nach dem Settlement einer Welle;
  (3) `unlock_time` im Täglich-Modus („5 neue Fragen")
- Auslöser 1 und 3 werden **zu einem Push verschmolzen**, wenn sie auf denselben Tag fallen —
  beide bedeuten „du hast offene Tipps". Hartes Limit von 2 Pushes pro Tag und Nutzer in einer
  `notifications_log`-Tabelle; bei vollem Limit fällt Auslöser 3 zuerst weg
- Deep Link aus dem Push direkt auf den Tipp-Screen des Events
- Rundenende: Siegerehrung Platz 1–3 mit Konfetti, geteilter Sieg explizit gefeiert,
  Teilen-Grafik für WhatsApp, **„Nächste Runde starten" für jedes Mitglied sichtbar**

**Verifiziert wenn:** Ein iOS-Gerät bekommt die Deadline-Erinnerung, nur wenn wirklich noch
Tipps offen sind; ein Tap öffnet den richtigen Screen. **Android bekommt in Expo Go keinen
Push** — das ist erwartet und dokumentiert, der *Tippen*-Tab ist dort das Sicherheitsnetz.

---

## Sprint 7 — Feinschliff und die echte Runde

- Light Mode auf allen Screens durchsehen (durch die Tokens ein Durchgang, kein Umbau)
- Empty States, Fehlerzustände, Offline-Banner, Ladeskelette
- Rechtstexte einbinden (AGB, Datenschutz, Impressum)
- KI-Illustrationen für Runden-Header, Siegerehrung und Share-Grafik
- **Erste echte Runde mit 8–10 Freunden**, gemischt über Sport, Finanzen und Politik

---

## Was ich zuerst brauche

1. **Supabase-Projekt** in EU Frankfurt (kann ich per MCP anlegen, sobald du zustimmst)
2. **API-Football-Key** (Free Tier) — erst ab Sprint 5 nötig
3. **Vercel-Projekt** fürs Admin-Web — erst ab Sprint 3 nötig

---

## Bewusst offen gelassen

- Fragetyp D (Ranking), Saison-Wertung, Badges, Premium, Apple/Google Sign-in, Store-Launch,
  Marketing-Website, KI-Generator, private Gruppen-Events — alles nach v1
- Das Schema bereitet `seasons` / `season_standings` vor, aber v1 berechnet dort nichts
