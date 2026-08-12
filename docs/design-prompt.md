# Agonmarket — Design-Prompt (Bildgenerierung)

> Prompt zur Erzeugung des kompletten UI-Design-Systems und aller Screens.
> Abgeleitet aus [spec.md](./spec.md). Bei Änderungen an Farben, Wording oder
> Bild-Strategie muss dieser Prompt mitgezogen werden.

---

Generate an image of a complete modern mobile app UI design system and all screen designs for a social prediction game app called **"Agonmarket"**.

Agonmarket is a competitive prediction game for closed groups of friends. Friends predict real-world outcomes from three worlds — **sport** (football matches: winner, exact score, over/under goals), **finance** (stocks, indices, crypto: up/down, above/below a threshold, price target) and **politics** (election percentages, outcomes) — spend a shared point budget on their predictions, and crown a champion at the end of each round.

**This is explicitly NOT a gambling app.** No money, no stakes with monetary value, no payouts, no odds, no bookmaker. Points are chips at a game night — they exist only inside a round and expire when it ends. The entire visual language must communicate *sports club leaderboard*, never *betting slip*.

The app helps users:
- Join a closed friend group via an invite link
- Create a round: duration in days, point budget, questions from a catalogue
- Predict outcomes and spend points strategically across questions
- See other members' predictions revealed only after the deadline
- Watch the leaderboard move live after each result
- Celebrate a champion at the end of the round

## UI language

**All interface text in German.** Use the real wording from the product: *tippen, Tipp, Punkte, Credits, Rangliste, Champion, Runde, Gruppe, Deadline, Auswertung*.

**Never** use these words anywhere, not even in placeholder text: *wetten, Wette, Quote, Einsatz, Gewinn, Jackpot, Bonus, Cashout*.

## Design Style

- Ultra modern 2026 startup aesthetic, in the spirit of Trade Republic, Sleeper and OneFootball
- **Dark mode first** — deep navy "arena at night" atmosphere, premium and focused
- Light mode as a **fully designed second variant**, not an afterthought (show key screens in both)
- Data-dense but calm: generous spacing, clear hierarchy, no visual noise
- Soft rounded corners (16–20px radius), subtle elevation, floating cards on a deep navy ground
- Pill-shaped buttons and category filter chips
- **Signature motif: very large, bold numerals** — point totals, countdowns, ranks — treated as the hero element of a screen rather than photography
- **Second signature motif: rank circles** in gold and mint, recurring across leaderboard, profile and awards — plain filled circles or rings, never segmented
- Competitive, sporty, confident — like a private league table, not a casino
- NOT cluttered, NOT corporate, NOT a betting site, NOT casino-styled. The closest reference points are a sports club league table and a modern brokerage app — **never a gaming floor**

## Absolute visual prohibitions

These are legal and brand constraints, not preferences:

- **No football club crests, no team logos, no league logos** (trademarks)
- **No press photography, no photos of real athletes, politicians or public figures**
- **No party logos or political figures**
- **NO CASINO IMAGERY OF ANY KIND — this is the single most important prohibition.** No roulette wheels, no roulette tables, no spinning wheels or wheel-of-fortune shapes, no gambling chips or tokens, no dice, no playing cards, no card suits, no slot machines, no jackpot or payout motifs, no coin piles, no felt-green surfaces, no neon casino signage, no lever or spin metaphors. **A radial segmented circle reads as a roulette wheel and must not appear** — including as an abstract decoration, a loading indicator, a progress ring or a background pattern. If a circular element is needed, use a plain rank circle or a simple ring, never one divided into coloured segments.
- **No currency symbols** (€, $) anywhere — points are never money
- **No odds displays** (2.45, 1/2, +150) — multipliers are shown as "×2", "×4" only
- No green/red "win/loss" betting-slip styling

## What replaces imagery

- **Category colour coding** as the primary identity of every question, always as a small tinted chip with a text label: Sport = mint, Finance = gold, Politics = soft blue `#8FB3F2`
- **Monogram circles**: two- or three-letter team abbreviations ("FCB", "RMA", "BVB") inside coloured circles using team-adjacent colours (colours are not trademarked)
- **Mini sparklines** for finance questions showing recent price movement
- **Crypto tickers and stock symbols** (BTC, ETH, DAX, AAPL) — these are fine
- **Flag emojis** for political questions — fine
- **Abstract AI-illustrated "arena night" artwork** for hero moments only: round headers, templates, the awards screen and the share graphic. Stylised stadium light cones, rising abstract charts, ballot box silhouettes, duel motifs. Geometric, atmospheric, no recognisable real people.
- Most list rows carry **no artwork at all** — lists stay quiet and fast

## Colour Palette

**Dark (default):**
- Background deep navy `#10182B`
- Card surface `#1B2740`
- **Electric violet `#7C5CFF` — the primary accent.** All primary CTAs, selected and active states, the user's own row, brand moments
- Signal mint `#02C39A` — **success only**: correct predictions, points won, upward movement
- Rank gold `#F2C14E` — **awards only**: first place, badges, champion, trophies
- Ice blue `#CADCFC` — secondary text

**The three accents are strictly semantic and never interchangeable.** Violet means *you can act / you are here*, mint means *this was right*, gold means *this was won*. A violet button is never a success signal, and a mint highlight is never a call to action.

**Category colours** are a separate scale and always appear as small tinted chips with a text label, never as bare colour bars, so they cannot be mistaken for a semantic signal: Sport `#02C39A`, Finance `#F2C14E`, Politics `#8FB3F2`.
- Pure white for primary text and large numerals

**Light (full second variant):**
- Background `#F4F7FB`
- Cards pure white with soft shadow
- Text `#1A2233`
- Violet, mint and gold stay **identical** as accents; only the violet used for text on white shifts one step darker to `#6A45E8` for legibility

Subtle violet glow on active and selected states, mint and gold reserved for results and awards. No gradients beyond soft atmospheric ones on hero artwork.

## Typography

- Clean modern sans-serif (SF Pro / Inter style)
- **Oversized numerals** as the brand signature: 48–72px point totals, countdowns and ranks, tabular figures
- Strong hierarchy: bold question headlines, medium labels, comfortable body
- Metadata pills: "Sport", "Finanzen", "Politik", "Noch 2h 14m", "exakt ×4 · Tendenz ×2", "25 Cr"
- Monospaced or tabular figures wherever numbers align in columns (leaderboards, budget bars)

---

## Screens

### 1. Splash Screen
Agonmarket logo centred on deep navy. Abstract arena-light artwork faintly behind. Minimal, premium.

### 2. Welcome Screen — two variants side by side
- **Without invite:** headline "Tippe mit deinen Freunden auf Sport, Finanzen & Politik", primary pill CTA "Loslegen"
- **With invite:** same screen plus a group context card at the top — "Du wurdest in ‚FC Feierabend' eingeladen" with group emoji and member avatars

### 3. Magic-Link Authentication — three states
- Email input, single field, label "E-Mail", pill CTA "Magic Link senden"
- "Prüf dein Postfach" confirmation showing the entered address, "Erneut senden" with a 60-second cooldown counter, "E-Mail ändern" link
- Error states: inline invalid-email validation, expired-link screen, offline banner with retry

**No password field anywhere. No separate sign-up path.** One flow for new and returning users.

### 4. Profile Setup (first login only)
Username field with live availability check (green mint check mark), avatar picker with abstract geometric presets, checkbox "Ich akzeptiere AGB & Datenschutzerklärung" with links, checkbox "Ich bin mindestens 18 Jahre alt". Continue button disabled until both are ticked.

### 5. Home — empty state and populated
- **Empty:** warm illustration, two clear paths as large cards: "Gruppe erstellen" and "Mit Code beitreten"
- **Populated:** list of groups, each row showing group emoji, name, member count, the active round, and a **violet** badge with the number of open predictions (it prompts an action, so it is not mint)

### 6. Create Group & Invite Sheet
Name field plus emoji picker. Immediately after creation, a bottom sheet opens: large invite code in oversized type, "Link kopieren" and a WhatsApp share button, headline "Lad deine Freunde ein".

### 7. Round Creation — 4-step flow with progress indicator
1. Round name, **mode selector as two large cards — "Täglich" (selected by default) and "Klassisch"** — duration chips 3 / 7 / 30 Tage / frei with **7 preselected**, and for daily mode two extra controls: questions per day (chips 3 / 5 / 7, **5** preselected) and unlock time (09:00). A helper line computes the total: "35 Fragen insgesamt"
2. Credit budget per participant, stepper, **default 1000**, hint "Max. 100 Credits pro Frage (10 %)"
3. Question picker with category filter chips (Sport / Finanzen / Politik), selectable question cards, template shortcuts "CL-Finale-Woche", "Krypto-Monat", "Wahl-Special" — plus a **"Beliebt diese Woche" section** at the top showing questions with a small popularity indicator (a count of active rounds, never group names)
4. Summary with optional ×2 weighting toggle per question, primary CTA "Los!"

### 8. THE QUESTION CARD — the central component, show all four states
This is the most important artboard. Render the same card in four states, in **both dark and light mode**:

- **Offen (open):** category chip, countdown "Noch 2h 14m", monogram circles "FCB" / "RMA", title "Bayern – Real", question "Wer gewinnt das Finale?", answer chips [1] [X] [2], scoring rule "exakt ×4 · Tendenz ×2", credit stepper showing "100 Cr"
- **Getippt (predicted):** own answer highlighted in **violet** with a saved check mark, plus **two visually distinct distribution bars stacked**: the group ("68% deiner Gruppe") in violet, and the app-wide aggregate ("54% aller Tipper") in a quieter secondary treatment — thinner, muted, clearly the less personal of the two. The second bar must read as context, never as a recommendation. Design an alternate version of this state **without** the app-wide bar, since it only appears once enough people have predicted
- **Gesperrt (locked):** deadline passed, all members' predictions revealed as a list of avatars grouped under each answer
- **Ausgewertet (settled):** result banner, points won per person, own row highlighted

Also show the input variants: chips for choice questions, two large buttons for over/under, a stepper for exact scores, a numeric field with a slider for percentages and prices.

### 9. Prediction Screen (event detail)
Event header with abstract arena artwork and a large countdown. Below it, all markets for that event as question cards. A persistent **budget bar** pinned at the top, always carrying **both numbers plus the remaining question count**: "640 von 1000 Credits · noch 20 Fragen offen". A cap hint appears when the 100-credit maximum per question is reached. Auto-save check marks. Text "Du kannst deinen Tipp bis zur Deadline ändern."

Also design the **daily unlock header** for rounds in daily mode: "Tag 3 von 7 · 5 neue Fragen", with the remaining days as a compact progress indicator.

### 10. "Tippen" Tab — the safety net
All open predictions across all groups, sorted by deadline. Section headers "Heute fällig", "Diese Woche". Each row shows group name, question, countdown and category colour. Header count "3 Tipps fällig heute".

### 11. Results Screen
Result banner with the actual outcome. Comparison list: who predicted what, who earned how many points. Own row marked with a **violet** identity bar on the left edge. Correct answers and points won in **mint**, incorrect in muted grey — **not red**. This screen is where the semantic split is most visible: violet says *this is you*, mint says *this was right*.

### 12. Leaderboard
Ranked list with rank circles: gold for 1st, mint for 2nd and 3rd, plain filled circles — never segmented. Large point numerals, with the split "gewonnene Punkte + Rest-Credits" shown as a smaller secondary line so the total is traceable. Movement arrows up and down since the last evaluation. Own row highlighted with a **violet** border. Header showing round name and remaining days.

### 13. Awards / Champion Screen
Celebratory moment. Podium for places 1–3 with oversized rank circles, confetti in mint and gold, abstract arena artwork behind. Champion's username in large type. Also design the **shared-victory variant** ("Doppelsieg!") with two names on the top step. Buttons: "Teilen" and "Nächste Runde starten" — the second visible to every member, not only the host.

### 14. Share Graphic
Square export card for WhatsApp: round name, top three with rank circles and point totals, Agonmarket wordmark, arena artwork background. Designed to be legible as a small chat thumbnail.

### 15. Push Permission Explainer
Shown after the first prediction, before the system dialog. Illustration, headline "Sollen wir dich vor Deadlines erinnern?", body explaining that reminders only arrive when predictions are still open, buttons "Ja, erinnern" and "Später".

### 16. Profile & Settings
Large circular avatar, username, stats in oversized numerals: Runden gespielt, Siege, Podiumsquote, Trefferquote, längste Serie. Badge shelf with locked and unlocked trophies. Settings rows: Benachrichtigungen, **Erscheinungsbild (System / Hell / Dunkel)**, Konto, Hilfe, and a muted "Account löschen" row.

### 17. States & Edge Cases
Empty states for: no groups, no open predictions, round not started, spectator mode ("Du bist Zuschauer bis zur nächsten Runde"). Plus an offline banner, a loading skeleton for the leaderboard, and an **annulled question card** — "Annulliert · Spiel abgesagt · Einsätze zurückerstattet".

### 18. Design System Sheet
One artboard documenting the system: colour tokens in both modes with hex values **and their semantic role labelled** (violet = action and identity, mint = success, gold = award), type scale including the oversized numeral role, buttons (primary violet pill, secondary outline, disabled), category chips, answer chips in unselected and selected state, the credit stepper, the budget bar with both numbers, rank circles in gold and mint, monogram circles, the sparkline component, and the countdown pill.

---

## Direction

Design this like a real App Store product from a European startup. Every screen must feel connected through one cohesive design system, and the dark and light variants must feel like the same product rather than two different apps.

The experience should feel: **competitive, sporty, premium, focused, social, confident, data-driven, celebratory at the right moments** — and never like gambling.

Generate high-quality mobile mockups in iPhone format with realistic spacing, German interface copy, oversized numerals as the visual signature, category colour coding instead of photography, and production-ready startup design quality.
