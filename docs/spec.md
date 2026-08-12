# Agonmarket — Produkt- & Architektur-Spec (v1)

> Stand: 12.08.2026 · Ergebnis des Gründungs-Interviews.
> Dieses Dokument hält fest **was** gebaut wird und **warum** die Regeln so lauten.
> Die Bau-Reihenfolge steht daneben in [implementation-plan.md](./implementation-plan.md).
>
> **Diese Datei ist die einzige Quelle der Wahrheit für Produktregeln.** Es gibt kein zweites
> Briefing-Dokument. Konzepttexte aus Chats oder Notizen sind Zulieferung — bei Widersprüchen
> gilt ausschließlich, was hier steht.

---

## Context

Warum es dieses Dokument gibt: Agonmarket wird von Grund auf neu gebaut (das Repo ist ein
leerer Expo-Blank-Start). Das Ausgangskonzept war ausführlich, enthielt aber vier Stellen, an
denen eine falsche Annahme später einen teuren Umbau erzwungen hätte — die Credit-Mechanik,
die Restcredit-Regel, die Content-Versorgung und den Settlement-Pfad. Alle vier sind geklärt und
unten festgehalten; die Restcredit-Regel wurde dabei bewusst in ihrer ursprünglichen Form
bestätigt, nachdem die Multiplikator-Konstruktion sie trägt (§ 2.2). Ziel: eine Mobile-App, mit
der geschlossene Freundesgruppen in zeitlich begrenzten Runden auf reale Ereignisse aus Sport,
Finanzen und Politik tippen — ausdrücklich ohne Geldwert, ohne Einsätze, ohne Auszahlung.

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
- Runden-Erstellung im 4-Schritt-Flow (Modus + Dauer, Credits, Fragen, Übersicht)
- **Rundenmodus „Täglich"** als Default: tägliche Freischaltung von Fragen (§ 2.5)
- Fragetypen **A** (Auswahl), **B** (Über/Unter), **C** (Exakter Wert)
- Alle drei Kategorien: **Sport, Finanzen, Politik**
- Tippen mit Credit-Einsatz, Auto-Save, änderbar bis Deadline
- Automatisches Settlement (Sport-API, Kurs-API) + manuelles Settlement (Politik)
- Live-Leaderboard, Siegerehrung
- Push: Deadline-Erinnerung + Auswertung
- **Dark Mode UND Light Mode**, vollständig, inkl. Umschalter + „System folgen"
- **Admin-Web (Next.js)** zum Anlegen und Auflösen von Events
- Account-Löschung (DSGVO), AGB/Datenschutz/18+-Bestätigung im Onboarding

### Schema in v1, Oberfläche später
Beide Features aus § 6 sind bei einer einzigen Testgruppe **wirkungslos** — „Beliebt diese Woche"
zeigt überall die Zahl 1, und die 20-Tipps-Schwelle der app-weiten Verteilung wird bei 8–10
Nutzern nie erreicht. Tabellen, RPC und Cron werden trotzdem in v1 mitgebaut, damit später nichts
umgebaut werden muss; die UI erscheint erst, wenn genug Nutzer die Schwellen füllen:
- **Trending-Fragen** im Katalog (§ 6.1)
- **App-weite Tipp-Verteilung** auf der Frage-Karte (§ 6.2)

### Explizit NICHT in v1
Fragetyp **D** (Ranking/Drag&Drop) · Saison-Wertung & Badges · Premium/RevenueCat ·
Apple/Google Sign-in · Store-Launch · Marketing-Website · KI-Fragen-Generator ·
private Gruppen-Events · Ranking-/Statistik-Tiefe im Profil

---

## 2. Entschiedene Regeln

### 2.1 Credit-Mechanik — Cap statt All-in
| Regel | Wert |
|---|---|
| Budget pro Teilnehmer und **gesamte Runde** | Host wählt, Default **1000** |
| Max. Einsatz pro Frage | **100 Credits = 10 % des Budgets** |
| Skalierung | Weicht der Host vom Default ab, bleiben es 10 % (Budget 500 → Cap 50) |
| Folge | Das volle Budget lässt sich auf **mind. 10 Fragen** verteilen |
| Pleite-Schutz | Restbudget < 5 Cr → Mindesteinsatz („Sockel") bleibt tippbar |

**Kein Tagesbudget.** Das Budget gilt für die gesamte Runde, auch im Täglich-Modus. Würden
Credits täglich neu zugeteilt, wäre „am letzten Tag alles setzen" die dominante Strategie, weil
ungenutzte Tagescredits sonst wertlos verfielen — genau der Effekt, den der Cap verhindern soll.

*Behebt:* „alles auf den Favoriten" war ohne Cap die dominante Strategie und ein Spieler konnte
an Tag 3 einer langen Runde pleite und damit für den Rest tot sein.

### 2.2 Restcredits zählen voll
**Endstand = gewonnene Punkte + nicht gesetzte Rest-Credits.** Die Rechnung ist auf dem
Leaderboard transparent ausgewiesen.

*Warum das trägt:* Weil der Multiplikator exakt der Optionsanzahl entspricht (§ 2.3), ist blindes
Raten **erwartungswertneutral** — bei `n` Optionen und Trefferquote `1/n` gilt `n × 1/n = 1`, der
Einsatz kommt im Mittel unverändert zurück. Ein Credit ist also genau so viel wert, wie ihn zu
raten. Daraus folgt:

- **Informiertes Tippen ist strikt positiv** — jede Trefferquote über `1/n` schlägt das Halten
- **Nichtstun entspricht dem Erwartungswert eines Zufallstippers**, nicht mehr
- Wer schlechter als der Zufall liegt, verliert gegen einen Spieler, der die App nie öffnet —
  das ist die bewusst akzeptierte Konsequenz dieser Regel

*Offen:* Ob die Balance in der Praxis stimmt, ist eine empirische Frage. **Nach dem M2-Test mit
echten Daten** wird geprüft, wie die tatsächlichen Trefferquoten zu den Multiplikatoren stehen.
Multiplikatoren und Cap sind bewusst einzelne Zahlen an einer Stelle — später leicht justierbar,
ohne Schema- oder Logikumbau.

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
  Im Täglich-Modus bedeutet das praktisch **nur bis zur ersten Auswertung an Tag 1**. Nach dem
  ersten realen Durchlauf ist zu prüfen, ob das zu streng ist.
- **Gleichstand Platz 1** → geteilter Sieg, in der Siegerehrung explizit gefeiert.

### 2.5 Rundenmodus

Jede Runde hat einen Modus. **Default ist „Täglich".**

| | Klassisch | **Täglich (Default)** |
|---|---|---|
| Freischaltung | alle Fragen sofort sichtbar | pro Tag eine Staffel, ab `unlock_time` |
| Fragen pro Tag | — | wählbar **3 / 5 / 7**, Default **5** |
| Standarddauer | frei | **7 Tage** |
| Fragen gesamt | frei | 7 × 5 = **35** bei Default |

Der Täglich-Modus ist der Grund, warum die App täglich geöffnet wird: Jeden Morgen gibt es neue
Fragen, statt einmal 35 Fragen abzuarbeiten und drei Tage später wiederzukommen.

**Regeln:**
- `unlock_time` ist eine Uhrzeit auf der Runde (Default 09:00 Europe/Berlin); daraus wird beim
  Erstellen für jede Frage ein konkretes `unlock_at` berechnet und festgeschrieben
- Vor `unlock_at` ist eine Frage weder sicht- noch tippbar — durchgesetzt per RLS, nicht im Client
- Das Budget bleibt **rundenweit** (§ 2.1), es gibt keine Tageszuteilung
- Eine Frage, deren Deadline vor ihrem `unlock_at` läge, darf nicht in die Runde aufgenommen
  werden — der Wizard muss das beim Zusammenstellen verhindern

---

## 3. Kernflows

**Onboarding (Ziel: erster Tipp < 60 s):** Welcome (mit Gruppenkontext bei Invite) → E-Mail →
Magic Link → Deep Link zurück in die App → Username + Avatar + AGB/18+-Checkbox → Landung.
Der Invite-Code wird **vor** dem Mailversand in `expo-secure-store` abgelegt und nach dem Login
eingelöst, damit er den Mail-Umweg überlebt. Push-Permission **nicht** im Onboarding, sondern
nach dem ersten abgegebenen Tipp mit eigenem Erklär-Screen.

**Kern-Loop (Täglich-Modus):** Morgens um `unlock_time` Push „5 neue Fragen" → Deep Link direkt
auf die Tagesstaffel → tippen (30 s) → vor Ablauf ggf. Erinnerung an offene Tipps → nach dem
Event Push „Auswertung ist da" → Ergebnis + Leaderboard mit Auf-/Absteiger-Pfeilen.

**Budget-Anzeige — verbindliches Format.** Überall, wo Credits angezeigt werden (Tipp-Screen,
Runden-Detail, *Tippen*-Tab), stehen **immer beide Zahlen plus die Restmenge an Fragen**:

> **640 von 1000 Credits · noch 20 Fragen offen**

Ohne die zweite Zahl kann niemand einschätzen, ob 640 Credits viel oder wenig sind — bei 2
verbleibenden Fragen sind sie ein Überschuss, bei 20 eine knappe Ration. Die Anzeige ist damit
das zentrale Steuerungsinstrument der Rundenstrategie, nicht bloß eine Statuszeile.

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

rounds              id, group_id, name, status,
                    mode(classic|daily) default 'daily',
                    duration_days default 7,
                    questions_per_day(3|5|7) default 5,      ← nur bei mode='daily'
                    unlock_time time default '09:00',        ← Europe/Berlin
                    credit_budget default 1000,
                    max_stake_per_question default 100       ← = 10 % des Budgets
round_events        round_id, event_market_id, weight(1|2),
                    unlock_at timestamptz                    ← bei 'classic' = Rundenstart
predictions         round_id, market_id, profile_id, answer(jsonb), stake      ← verdeckt via RLS
results             market_id, outcome(jsonb), settled_at, settled_by
scores              round_id, market_id, profile_id, payout                    ← idempotent

  ── Aggregate (§ 6), enthalten keinerlei Personen- oder Gruppenbezug ──
trending_markets           event_market_id, active_round_count   ← Materialized View
market_distribution_cache  market_id, answer_bucket, cnt, total, refreshed_at
```

`stake_cap_pct` entfällt — der Cap steht als absoluter Wert `max_stake_per_question` auf der
Runde und wird beim Erstellen aus `credit_budget × 10 %` vorbelegt.

**Wichtig:** `events`/`event_markets` sind **global** — „Bayern – Real" existiert einmal und wird
einmal ausgewertet. Settlement-Kosten und API-Calls skalieren damit mit der Anzahl Events, nicht
mit der Anzahl Nutzer oder Gruppen.

**Verdeckte Tipps** werden per RLS erzwungen, nicht im Client: fremde `predictions` sind erst
lesbar, wenn `now() > settlement_at` bzw. der Markt gesperrt ist. Diese Policy bleibt auch mit
den Aggregaten aus § 6 **unverändert streng** — kein Client liest je fremde Einzeltipps.

**Noch nicht freigeschaltete Fragen** werden ebenfalls per RLS ausgeblendet: `round_events` mit
`unlock_at > now()` sind für Mitglieder nicht lesbar. Sonst könnte ein Client die kommenden
Tagesfragen vorab aus der API ziehen.

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

**Push-Regeln:** drei Auslöser, hartes Limit weiterhin **2 Pushes/Tag/Nutzer**.

| Priorität | Auslöser | Empfänger |
|---|---|---|
| 1 | T-2h vor der frühesten offenen Deadline | nur Nicht-Tipper, pro Runde gebündelt |
| 2 | nach dem Settlement einer Welle | alle Teilnehmer der Runde |
| 3 | `unlock_time` im Täglich-Modus: „5 neue Fragen" | alle Teilnehmer der Runde |

Weil der Täglich-Modus drei tägliche Auslöser erzeugt, das Limit aber zwei bleibt: **Freischaltung
und Deadline-Erinnerung werden zu einem Push verschmolzen**, wenn sie auf denselben Tag fallen —
beide sagen inhaltlich „du hast offene Tipps" („5 neue Fragen · 20 Fragen offen"). Läuft das
Limit trotzdem voll, entscheidet die Priorität oben; Nummer 3 fällt zuerst weg, weil die
Freischaltung ohnehin beim nächsten Öffnen der App sichtbar ist.

**Bild-Strategie:** keine Wappen, keine Pressefotos, keine Politiker-Bilder. Stattdessen
Kategorie-Chips, Icons, Monogramm-Kreise und Sparklines; KI-generierte Arena-Night-Illustrationen
für große Momente.

**Casino-Verbot (härteste Gestaltungsregel):** keine Roulette-Räder, Glücksräder, Chips, Würfel,
Spielkarten, Kartensymbole, Automaten, Jackpot-Motive, Münzstapel oder Filzgrün — weder als
Illustration noch als Dekoration, Ladeanzeige, Fortschrittsring oder Hintergrundmuster.
**Ein radial segmentierter Kreis liest sich als Roulette-Rad und ist damit ausgeschlossen;**
Rang-Kreise sind immer einfarbig gefüllte Kreise oder schlichte Ringe. Die Referenzpunkte sind
Ligatabelle und Broker-App, nie ein Spielsaal.

**Design-Tokens** (semantisch, nie rohe Hex-Werte in Komponenten):

| Token | Dark | Light | Bedeutung |
|---|---|---|---|
| `background` | `#10182B` | `#F4F7FB` | Grundfläche |
| `surface` | `#1B2740` | `#FFFFFF` | Karten |
| `textPrimary` | `#FFFFFF` | `#1A2233` | Fließtext, große Zahlen |
| `textMuted` | `#CADCFC` | `#5B6B85` | Sekundärtext |
| **`accent`** | **`#7C5CFF`** | **`#6A45E8`** | **Primärakzent: Aktionen, Auswahl, „das bist du"** |
| `success` | `#02C39A` | `#02C39A` | richtig getippt, gewonnene Punkte, Aufstieg |
| `award` | `#F2C14E` | `#F2C14E` | Platz 1, Badges, Champion |
| `categorySport` | `#02C39A` | `#02C39A` | Kategorie-Chip |
| `categoryFinance` | `#F2C14E` | `#F2C14E` | Kategorie-Chip |
| `categoryPolitics` | `#8FB3F2` | `#8FB3F2` | Kategorie-Chip |

**Die drei Akzente sind strikt semantisch und nie austauschbar:** Violett heißt *du kannst
handeln / das bist du*, Mint heißt *das war richtig*, Gold heißt *das wurde gewonnen*. Ein
violetter Button ist nie ein Erfolgssignal, ein Mint-Highlight nie ein Call-to-Action.

**Kategorie-Farben teilen bewusst Werte mit `success` und `award`.** Damit daraus keine
Verwechslung wird, erscheinen Kategorien **ausschließlich als kleine getönte Chips mit Textlabel**
(„Sport", „Finanzen", „Politik"), nie als reine Farbflächen oder Balken. Die Unterscheidung läuft
über die Form, nicht über den Farbton. `categoryPolitics` ist deshalb auch **nicht** identisch mit
`textMuted` — sonst wäre ein Politik-Chip von deaktiviertem Text nicht zu unterscheiden.

**Wording überall:** tippen, Punkte, Rangliste, Champion. Nie: wetten, Einsatz, Quote, Gewinn.

---

## 6. Aggregate über Gruppengrenzen hinweg

Zwei Features zeigen Zahlen, die aus den Daten **aller** Nutzer entstehen. Beide folgen demselben
Muster: **vorberechnetes Aggregat + kontrollierte Lesefunktion**, niemals eine Live-Abfrage auf
`predictions` und niemals ein gelockertes RLS.

### 6.1 Trending-Fragen im Wizard

Schritt 3 der Runden-Erstellung bekommt eine Sektion **„Beliebt diese Woche"**: Fragen sortiert
nach der Anzahl aktiver Runden, in denen sie gerade läuft.

- **Ebene:** der Zähler hängt an **`event_markets`**, nicht an `events`. Der Wizard wählt Fragen
  (= Märkte), und „Bayern – Real" hat drei Märkte, die unterschiedlich beliebt sein können.
- **Aggregation:** `count(distinct round_id)` über `round_events ⋈ rounds` mit
  `rounds.status = 'active'`.
- **Form:** **Materialized View** `trending_markets`, per `pg_cron` alle 15 Minuten mit
  `REFRESH ... CONCURRENTLY` (braucht einen Unique-Index). Bewusst *keine* trigger-gepflegte
  Zählerspalte: „aktiv" ändert sich, wenn eine **Runde endet**, ohne dass `round_events`
  angefasst wird — Trigger müssten zusätzlich auf `rounds.status` feuern und driften still
  auseinander, sobald einer davon fehlschlägt. Die View wird stattdessen komplett neu berechnet.
- **Datenschutz:** die View enthält ausschließlich `(event_market_id, active_round_count)`. Kein
  `round_id`, kein `group_id` — der Gruppenbezug existiert im Aggregat gar nicht, statt nur nicht
  abgefragt zu werden.
- **Anzeigeschwelle:** erst ab **3 aktiven Runden**, sonst ist die Sektion eine Liste von Fragen
  mit dem Zähler 1.
- **Supabase-Hinweis:** Materialized Views kennen kein RLS und werden in `public` automatisch über
  PostgREST exponiert. Da der Inhalt bewusst anonym ist, ist das vertretbar — sauberer ist ein
  privates Schema mit Zugriff über eine Funktion, sonst meldet der Security-Advisor ein Finding.

### 6.2 App-weite Tipp-Verteilung auf der Frage-Karte

Nach dem eigenen Tipp zeigt die Karte zusätzlich zur Gruppenverteilung die Verteilung aller
Agonmarket-Nutzer: **„54 % aller Tipper"**.

- **Zugriffsweg:** eine **`SECURITY DEFINER`-Funktion** `get_market_distribution(market_id)`,
  gehärtet mit `SET search_path = ''`, vollqualifizierten Namen,
  `REVOKE EXECUTE FROM public, anon` und `GRANT EXECUTE TO authenticated`.
- **Kein Service-Role-Key im Client — niemals.** Der Key umgeht sämtliches RLS und ein
  Expo-Bundle ist trivial auslesbar; damit läge jede fremde Vorhersage, jede Gruppe und jedes
  Profil offen. `SECURITY DEFINER` erreicht dieselbe Wirkung, umgeht RLS aber an genau einer
  auditierbaren Stelle statt im Client.
- **RLS bleibt unverändert streng.** Clients lesen weiterhin keine fremden `predictions`.
- **Eigener-Tipp-Sperre in der Funktion**, nicht im UI: ohne eigenen Tipp liefert die RPC nichts.
  Sonst holt sich ein Client das Schwarmsignal, ohne sich festzulegen.
- **Schwellen gegen De-Anonymisierung:**
  - Gesamtzahl der Tipps **< 20** → keine Ausgabe
  - Typ A/B (2–4 Optionen): Verteilung über alle Optionen
  - **Typ C** (exakte Werte): Der Antwortraum ist praktisch unbegrenzt, 20 Tipps können sich auf
    15 Antworten mit `n = 1` verteilen. Deshalb nur **Top 3 + „Sonstige"**, und jeder Eimer unter
    `n = 5` wird unterdrückt.
- **Gegen zeitliche Differenzbildung:** Tipps sind bis zur Deadline änderbar. Ein Client, der die
  RPC im Minutentakt abfragt, sähe an den Deltas knapp über der Schwelle einzelne Personen. Die
  RPC liest deshalb **nicht live**, sondern aus `market_distribution_cache`, das `pg_cron` alle
  5 Minuten für offene Märkte neu berechnet. Das vergröbert die Deltas und löst zugleich das
  Kostenproblem: `predictions` wird die größte Tabelle, und eine Live-Aggregation bei jedem
  Rendern einer Frage-Karte wäre der teuerste Query der App.
- **Niemals** Einzeltipps oder Namen außerhalb der eigenen Gruppe — auch nicht nach der Deadline.

---

## 7. ASSUMPTIONS (von mir gesetzt, bitte bestätigen oder korrigieren)

1. ~~**Cap bei kleinen Runden**~~ **HINFÄLLIG:** Die Sonderformel
   `max(25 % × Budget, Budget / Fragenanzahl)` existierte nur, damit sich das Budget in kleinen
   Runden vollständig ausgeben ließ — sonst wären Restcredits ersatzlos verfallen. Da Restcredits
   jetzt **voll zählen** (§ 2.2), entsteht durch nicht ausgebbare Credits kein Nachteil mehr.
   Der Cap ist damit schlicht `max_stake_per_question`, ohne Fallback.
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
8. **Balance-Check nach M2:** Multiplikatoren (§ 2.3), Cap und Budget (§ 2.1) sind
   Erfahrungswerte. Nach dem M2-Test mit echten Daten wird geprüft, wie sich die tatsächlichen
   Trefferquoten zu den Multiplikatoren verhalten — insbesondere, wie viele Spieler unter dem
   Zufallsniveau liegen und damit gegen einen Nicht-Tipper verlieren. Alle drei Größen sind
   einzelne Zahlen an einer Stelle und ohne Schema- oder Logikumbau justierbar.

   **Wenn sich Nicht-Tippen dann zu gut anfühlt, ist der Hebel NICHT die Restcredit-Regel.**
   In dieser Reihenfolge nachjustieren:
   1. **Multiplikatoren** anheben (§ 2.3) — die Regel bleibt, informiertes Tippen wird lohnender
   2. **Cap oder Budget** anpassen (§ 2.1)
   3. **Kleiner Teilnahme-Bonus** pro abgegebenem Tipp — bewusst als dritte Option offengehalten,
      weil sie die Erwartungswert-Logik von § 2.2 unangetastet lässt und trotzdem Aktivität belohnt

   § 2.2 selbst wieder umzudrehen wäre der teuerste Eingriff, weil daran Leaderboard-Darstellung,
   Siegerehrung und die gesamte Testsuite hängen.

---

## 8. OPEN RISKS

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
7. **Fragen-Nachschub ist der eigentliche Dauerlauf — und der Täglich-Modus verfünffacht ihn.**
   Eine Standardrunde braucht **35 auflösbare Märkte pro Woche** (7 Tage × 5 Fragen), jeder mit
   Quelle, Regel und Stichtag, die Politik-Fragen zusätzlich manuell von dir aufgelöst. Die
   ursprüngliche Referenz war ein Bundesliga-Spieltag mit ~13 Märkten. Damit verschiebt sich der
   **KI-Fragen-Generator von „später" zu „notwendig"**: nebenberuflich 35 Fragen pro Woche von
   Hand zu kuratieren und aufzulösen ist der Punkt, an dem der Betrieb als Erstes reißt. Die
   Freigabe-Queue im Admin-Web (Assumption 2) ist deshalb kein Komfort, sondern die Vorbereitung
   darauf.
