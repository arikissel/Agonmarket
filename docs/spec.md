# Agonmarket — Produkt- & Architektur-Spec (v1)

> Stand: 12.08.2026 · Ergebnis des Gründungs-Interviews.
> Dieses Dokument hält fest **was** gebaut wird und **warum** die Regeln so lauten.
> Die Bau-Reihenfolge steht daneben in [implementation-plan.md](./implementation-plan.md).

---

## Context

Warum es dieses Dokument gibt: Agonmarket wird von Grund auf neu gebaut (das Repo ist ein
leerer Expo-Blank-Start). Das Ausgangskonzept war ausführlich, enthielt aber vier Stellen, an
denen eine falsche Annahme später einen teuren Umbau erzwungen hätte — die Credit-Mechanik,
die Restcredit-Regel, die Content-Versorgung und den Settlement-Pfad. Diese sind im Interview
geklärt und unten festgehalten. Ziel: eine Mobile-App, mit der geschlossene Freundesgruppen in
zeitlich begrenzten Runden auf reale Ereignisse aus Sport, Finanzen und Politik tippen —
ausdrücklich ohne Geldwert, ohne Einsätze, ohne Auszahlung.

---

## 1. Produkt & Scope

**Kernsatz:** Kicktipp trifft die Themenbreite eines Prediction Markets, verpackt als
kompetitiver digitaler Stammtisch für geschlossene Freundeskreise.

**Nutzer:** Bestehende WhatsApp-Gruppen (Freunde, Kollegen, Verein). Einstieg fast immer über
einen geteilten Invite-Link, nicht über den Store.

**v1 ist fertig, wenn:** Eine vollständige Runde — Erstellung → Tippen → Deadline → Auswertung
→ Leaderboard → Siegerehrung — von 8–10 echten Freunden über **Expo Go** durchgespielt wurde.

### In Scope (v1)
- Magic-Link-Login (E-Mail), ein Flow für Neu und Bestand
- Gruppen (Invite-Code/-Link), Rollen Host/Member
- Runden-Erstellung im 4-Schritt-Flow (Dauer, Credits, Fragen, Übersicht)
- Fragetypen **A** (Auswahl), **B** (Über/Unter), **C** (Exakter Wert)
- Alle drei Kategorien: **Sport, Finanzen, Politik**
- Tippen mit Credit-Einsatz, Auto-Save, änderbar bis Deadline
- Automatisches Settlement (Sport-API, Kurs-API) + manuelles Settlement (Politik)
- Live-Leaderboard, Siegerehrung
- Push: Deadline-Erinnerung + Auswertung
- **Dark Mode UND Light Mode**, vollständig, inkl. Umschalter + „System folgen"
- **Admin-Web (Next.js)** zum Anlegen und Auflösen von Events
- Account-Löschung (DSGVO), AGB/Datenschutz/18+-Bestätigung im Onboarding

### Explizit NICHT in v1
Fragetyp **D** (Ranking/Drag&Drop) · Saison-Wertung & Badges · Premium/RevenueCat ·
Apple/Google Sign-in · Store-Launch · Marketing-Website · KI-Fragen-Generator ·
private Gruppen-Events · Ranking-/Statistik-Tiefe im Profil

---

## 2. Entschiedene Regeln (die vier kritischen Korrekturen)

### 2.1 Credit-Mechanik — Cap statt All-in
| Regel | Wert |
|---|---|
| Budget pro Teilnehmer und Runde | Host wählt, Default **100** |
| Max. Einsatz pro Frage | **25 % des Budgets** |
| Folge | Budget muss auf **mind. 4 Fragen** gestreut werden |
| Pleite-Schutz | Restbudget < 5 Cr → Mindesteinsatz („Sockel") bleibt tippbar |

*Behebt:* „alles auf den Favoriten" war die dominante Strategie und ein Spieler konnte an
Tag 3 einer 30-Tage-Runde pleite und damit für den Rest tot sein.

### 2.2 Restcredits verfallen
**Endstand = ausschließlich gewonnene Punkte.** Nicht gesetzte Credits sind am Rundenende
wertlos.

*Behebt:* Nach der Originalformel (`Punkte + Restcredits`) hätte ein Spieler, der **gar nicht
tippt**, mit 100 Punkten oft vor aktiven Spielern gelegen — der Passivste hätte gewonnen.

### 2.3 Multiplikatoren (fest, transparent, keine Quoten)
| Typ | Wertung |
|---|---|
| A / B | richtig = Einsatz × Anzahl Optionen (2 Optionen → ×2, 3 Optionen → ×3), falsch = 0 |
| C (Spielergebnis) | exakt ×4 · richtige Tendenz ×2 · sonst 0 |
| C (Prozent/Kurs) | ±1 → ×4 · ±3 → ×2 · sonst 0 |
| Gewichtung | Host kann einzelne Fragen ×2 zählen lassen |

Die Staffel steht sichtbar auf jeder Frage-Karte.

### 2.4 Ausfall & Streit
- **Nicht auflösbar** (Spiel abgesagt, API stumm, Ergebnis mehrdeutig) → Frage **annulliert**,
  alle Einsätze zurück ins Budget, Frage zählt nicht für die Wertung. Braucht eine
  Reversal-Logik im Settlement-Worker.
- **Letzte Instanz:** immer der Plattform-Admin (du), nie der Host. Auflösungsquelle und -regel
  stehen **unveränderlich ab Veröffentlichung** an der Frage.
- **Beitritt zur laufenden Runde** nur, solange keine Frage ausgewertet wurde; danach Zuschauer.
- **Gleichstand Platz 1** → geteilter Sieg, in der Siegerehrung explizit gefeiert.

---

## 3. Kernflows

**Onboarding (Ziel: erster Tipp < 60 s):** Welcome (mit Gruppenkontext bei Invite) → E-Mail →
Magic Link → Deep Link zurück in die App → Username + Avatar + AGB/18+-Checkbox → Landung.
Der Invite-Code wird **vor** dem Mailversand in `expo-secure-store` abgelegt und nach dem Login
eingelöst, damit er den Mail-Umweg überlebt. Push-Permission **nicht** im Onboarding, sondern
nach dem ersten abgegebenen Tipp mit eigenem Erklär-Screen.

**Kern-Loop:** Push „Deadline in 2h" → Deep Link direkt auf den Tipp-Screen → tippen (30 s) →
nach dem Event Push „Auswertung ist da" → Ergebnis + Leaderboard mit Auf-/Absteiger-Pfeilen.

**Rundenende:** Siegerehrung (Platz 1–3, Konfetti) → Teilen-Grafik → **jedes Mitglied** kann die
nächste Runde starten (nicht nur der Host — verhindert, dass die Gruppe an einer Person hängt).

**Navigation:** Bottom-Tabs — *Home* (Gruppen), *Tippen* (alle offenen Tipps nach Deadline
sortiert, das Sicherheitsnetz für alle, die Pushes ignorieren), *Profil*.

---

## 4. Datenmodell (Kern)

```
profiles            id, username(uniq), avatar, push_token, tos_accepted_at, age_confirmed_at
groups              id, name, emoji, invite_code(uniq), created_by
group_members       group_id, profile_id, role(host|member), joined_at

events              id, category(sport|finance|politics), title, starts_at        ← GLOBAL
event_markets       id, event_id, type(A|B|C), question, options,
                    settlement_source, settlement_rule, settlement_at,
                    settlement_type(api_auto|manual), status(open|locked|settled|voided)

rounds              id, group_id, name, duration_days, credit_budget,
                    stake_cap_pct(25), status
round_events        round_id, event_market_id, weight(1|2)
predictions         round_id, market_id, profile_id, answer(jsonb), stake      ← verdeckt via RLS
results             market_id, outcome(jsonb), settled_at, settled_by
scores              round_id, market_id, profile_id, payout                    ← idempotent
```

**Wichtig:** `events`/`event_markets` sind **global** — „Bayern – Real" existiert einmal und wird
einmal ausgewertet. Settlement-Kosten und API-Calls skalieren damit mit der Anzahl Events, nicht
mit der Anzahl Nutzer oder Gruppen.

**Verdeckte Tipps** werden per RLS erzwungen, nicht im Client: fremde `predictions` sind erst
lesbar, wenn `now() > settlement_at` bzw. der Markt gesperrt ist.

---

## 5. Architektur

| Ebene | Technologie |
|---|---|
| App | Expo **SDK 57.0.12** · React Native **0.86.2** · React **19.2.3** · TypeScript 6 · expo-router · **StyleSheet + Design-Tokens** (kein NativeWind, s. u.) |
| State | React Query + Zustand + supabase-js |
| Backend | Supabase (EU Frankfurt): Postgres + RLS, Auth (Magic Link), Realtime, Edge Functions |
| Settlement | Idempotenter Worker via pg_cron / Edge Function |
| Sport | API-Football (Free Tier) |
| Kurse | CoinGecko + Twelve Data (Free Tier) |
| Push | expo-notifications, Token in `profiles` |
| Admin | Next.js auf Vercel, geschützt |
| Verteilung v1 | **Expo Go** |

**Stichtags-Regel Finanzen:** ein einziger Auswertungszeitpunkt **22:00 Europe/Berlin**.
Krypto = Kurs zu genau diesem Moment; Aktien/Indizes = letzter verfügbarer Schlusskurs
(Wochenende/Feiertag → letzter Handelstag). Keine Börsenkalender-Logik.

**Push-Regeln:** genau zwei Auslöser — (1) T-2h vor der frühesten offenen Deadline einer Runde,
**nur an Nicht-Tipper**, pro Runde gebündelt; (2) nach dem Settlement einer Welle. Hartes Limit
2 Pushes/Tag/Nutzer.

**Bild-Strategie:** keine Wappen, keine Pressefotos, keine Politiker-Bilder. Stattdessen
Kategorie-Farbcode (Sport = Mint, Finanzen = Gold, Politik = Ice Blue), Icons,
Monogramm-Kreise und Sparklines; KI-generierte Arena-Night-Illustrationen für große Momente.

**Design-Tokens** (semantisch, nie rohe Hex-Werte in Komponenten):
Deep Navy `#10182B` · Card Navy `#1B2740` · Signal Mint `#02C39A` · Rank Gold `#F2C14E` ·
Ice Blue `#CADCFC` · Light: Fläche `#F4F7FB`, Karten Weiß, Text `#1A2233`.

**Wording überall:** tippen, Punkte, Rangliste, Champion. Nie: wetten, Einsatz, Quote, Gewinn.

---

## 6. ASSUMPTIONS (von mir gesetzt, bitte bestätigen oder korrigieren)

1. **Cap bei kleinen Runden:** Bei weniger als 4 Fragen wäre ein 25-%-Cap unspielbar (bei
   1 Frage könnten nur 25 von 100 Credits gesetzt werden, der Rest verfällt für alle gleich).
   Angenommene Formel: `Cap = max(25 % × Budget, aufgerundet Budget / Fragenanzahl)`.
2. **KI-Generator-Reihenfolge:** Das Admin-Web bekommt in v1 manuelle Erfassung **plus eine
   Freigabe-Queue**. Der KI-Generator (Polymarket Gamma als Ideenquelle) schreibt später in
   dieselbe Queue — kein Umbau, nur eine spätere Ausbaustufe.
3. **Rechtstexte** (AGB, Datenschutz, Impressum) kommen aus einem Generator/Anwalt, nicht von
   mir; ich baue nur die Einbindung und speichere den Zustimmungszeitpunkt.
4. **Sprache:** v1 ausschließlich Deutsch, Strings aber zentral abgelegt.
5. **Live-Leaderboard:** React-Query-Polling als Basis, Supabase Realtime nur für die
   Leaderboard-Ansicht. Kein Realtime-Abo für Listen.
6. **Gruppengröße** in v1 technisch unbegrenzt (Free-Tier-Limit „2 Gruppen" gehört zu Premium
   und kommt später).
7. **Saison-Wertung** (10/6/4/1 Punkte) wird im Schema vorbereitet, aber in v1 nicht berechnet
   oder angezeigt.

---

## 7. OPEN RISKS

1. **Scope vs. Zeit.** v1 wurde als „eine Runde mit Freunden" definiert, gleichzeitig wurden
   Admin-Web (zweite Codebase), beide Farbmodi vollständig und alle drei Settlement-Pfade
   gewählt. Nebenberuflich solo bedeutet das realistisch **8–10 statt 3–4 Wochen** bis zur
   ersten gespielten Runde. Jeder dieser drei Punkte ließe sich nachträglich zuschalten.
2. **Expo Go blockiert den Engagement-Loop.** Android bekommt seit SDK 53 **keine Push-Nachrichten
   in Expo Go** (iOS funktioniert). Der Custom-Scheme-Deeplink `agonmarket://` braucht ebenfalls
   einen echten Build; in v1 läuft Auth über `exp://`. Push wird gebaut, ist aber in v1 nur auf
   iOS überprüfbar — Android-Tester nutzen den *Tippen*-Tab. Umstieg später = ein EAS-Befehl.
3. ~~**NativeWind × React Native 0.86 ist nicht verifiziert.**~~ **GEKLÄRT (12.08.2026):**
   NativeWind v5 ist laut eigener Doku ausdrücklich *„pre-release, not intended for production
   use"* (Anleitung noch auf SDK 54, erzwungenes `lightningcss@1.30.1`-Pinning); NativeWind
   v4.2.6 deklariert **keinerlei `react-native`-Peer-Dependency**, also keine Aussage zu RN 0.86
   oder zur seit RN 0.82 standardmäßigen neuen Architektur. **Entscheidung: kein NativeWind.**
   Stattdessen `StyleSheet` über ein typisiertes Token-Modul mit `useTheme()`/`useStyles()`.
   Kein Babel-Plugin, kein Metro-Transformer, kein Pinning — und Dark/Light wird zum Tausch
   eines Token-Sets statt zu `dark:`-Varianten auf jeder Komponente.
4. **Politik-Settlement ist Handarbeit ohne Vertretung.** Jede Politik-Frage muss von dir manuell
   aufgelöst werden. Bist du im Urlaub, steht das Leaderboard der Gruppe still.
5. **18+ vs. „kein Glücksspiel".** Bewusste Entscheidung, in v1 kostenlos (kein Store-Review).
   Ab Store-Launch: höheres Age Rating, erhöhte Reviewer-Aufmerksamkeit Richtung Glücksspiel und
   Wegfall der Schul-/Uni-Zielgruppe, die Kicktipp groß gemacht hat.
6. **API-Free-Tiers.** API-Football Free liegt bei ~100 Requests/Tag. Für eine Gruppe reicht das
   klar, aber Spielplan-Abruf und Ergebnis-Polling müssen von Anfang an gebatcht und gecacht
   werden — nachträglich einzubauen wäre teurer.
7. **Fragen-Nachschub ist der eigentliche Dauerlauf.** Ohne wöchentlich frische Fragen stirbt die
   App unabhängig von der Codequalität. Der KI-Generator ist damit kein Nice-to-have, sondern der
   Punkt, an dem sich mittelfristig entscheidet, ob das Produkt trägt.
