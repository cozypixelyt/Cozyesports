# COZY_ESPORTS - Multi-Game Esports Tournament Platform

A high-octane esports tournament platform inspired by Stan, tailored for mobile and desktop gamers. Users can register for tournaments across **BGMI**, **Free Fire**, and **Minecraft**, recharge Cozy Coins via Admin UPI QR code submission, and receive real-time Room ID and Passwords once registered. An integrated Developer/Admin mode protected by a secret PIN allows the organizer to create tournaments, manage slots, approve coin top-ups, and broadcast room credentials on the fly.

---

### User Review & Critical Decisions

> [!IMPORTANT]
> The following architectural decisions have been aligned based on your clarifying answers:

- **Coin Recharge & Wallet Mechanism**: Players recharge by viewing the Admin UPI QR code, paying via their preferred UPI app (PhonePe, Google Pay, Paytm), and submitting their Transaction ID (UTR). The organizer approves requests in the Developer Panel, instantly crediting coins to the player's wallet.
- **Developer/Admin Mode Access**: Protected by a secret Developer PIN (`7788`, customizable in settings) accessible via a lock icon in the top header. Unlocking reveals the full in-app Tournament Creator, Coin Top-up Request Manager, and Room ID/Password broadcaster.
- **Tournament Card Details**: Cards display game tags, match schedule countdown, entry fee in Cozy Coins, prize pool breakdown, dynamic slot progress bar (e.g., `34/48 Joined`), match format (Solo/Duo/Squad), and the protected Room ID & Password section (unlocked only for registered players).

---

### 1. Overview & Core Concept

- **What It Does**: Gamers can browse tournaments categorized by game (BGMI, Free Fire, Minecraft, and others), filter by format (Solo, Duo, Squad) or status (Upcoming, Live, Completed), join tournaments by paying Cozy Coins, and view match room details. The admin/developer can manage the entire platform within the application without needing code modifications.
- **Target Audience**: Mobile and PC competitive gamers seeking structured daily scrims and prize tournaments, alongside esports tournament organizers.
- **Key Value**: Eliminates disorganized WhatsApp/Discord tournament registration spreadsheets with a streamlined, automated, and secure in-app ecosystem.

---

### 2. User Experience & Visual Design

- **Aesthetic Direction**: Deep dark esports arena aesthetic (`#090d16` canvas, `#131926` structural surfaces, `#1e293b` hairline borders). Accents in electric amber (`#f59e0b`) and cyber cyan (`#06b6d4`) for high-visibility gaming metrics.
- **Typography & Scale**: 
  - Brand & Hero: Condensed athletic sans (`Big Shoulders Display` / `Outfit` / `Barlow Condensed`) with high visual energy.
  - Body & UI: `Plus Jakarta Sans` for crisp UI legibility.
  - Match Data & Numbers: Tabular monospace (`font-mono tabular-nums`) for prize pools, entry fees, and slot counters.
- **Navigation Architecture**:
  - **Top Bar**: 1-row header with brand wordmark `COZY_ESPORTS`, active Cozy Coin balance pill with direct Top-up action, and the secret Developer Lock button.
  - **Game Selector Strip**: Filter tabs for All Games, BGMI, Free Fire, Minecraft, and Special Events.
  - **Main Views**:
    1. **Tournaments Feed**: Filterable grid of active matches with live slot indicators.
    2. **Tournament Details Modal/Drawer**: Full match rules, prize split (1st, 2nd, 3rd, per-kill), player roster, and Room ID/Password reveal module.
    3. **Coin Recharge Modal**: Admin UPI QR code generator, copyable UPI ID, amount selection presets (₹50, ₹100, ₹200, ₹500), and transaction ID verification submission.
    4. **My Matches Tab**: Shows registered tournaments with live countdowns and room credentials.
    5. **Developer Dashboard**: Instant tournament creation wizard, pending coin top-up approvals, participant list inspection, and room credentials updater.

---

### 3. Key Product Decisions & Trade-Offs

- **State Management & Persistence**:
  - *Chosen Approach*: Reactive local state engine with `localStorage` persistence, pre-seeded with realistic tournament data for BGMI, Free Fire, and Minecraft.
  - *Why*: Enables instant zero-latency testing, survives page refreshes, and lets the developer immediately create and manage tournaments right in the browser preview.
- **Room Credentials Security**:
  - *Chosen Approach*: Client-side verification where Room ID and Room Password are masked with a lock barrier unless the current user's profile ID is listed in the tournament's registered participant roster.
  - *Why*: Matches real-world esports scrim workflows where only verified paid entries receive the custom room credentials.
- **Coin Approval Pipeline**:
  - *Chosen Approach*: User submits payment with their In-Game Name (IGN) and UTR. Request is queued into `pendingPayments`. Developer can 1-click Approve (which adds coins and logs a transaction) or Reject with notes. Admin can also directly inject coins into any player wallet.

---

### 4. Technical Architecture & Data Strategy

```
┌────────────────────────────────────────────────────────────────────────┐
│                              COZY_ESPORTS                              │
├────────────────────────────────┬───────────────────────────────────────┤
│           Top Bar              │  Wordmark · Coin Balance · Dev Unlock  │
├────────────────────────────────┴───────────────────────────────────────┤
│                                                                        │
│   ┌───────────────────────────────────────────────────────────────┐    │
│   │ Game Filter Bar (All Games · BGMI · Free Fire · Minecraft)   │    │
│   └───────────────────────────────────────────────────────────────┘    │
│                                                                        │
│   ┌───────────────────────────────┐   ┌───────────────────────────┐    │
│   │      Tournament Cards Feed    │   │      Active User Panel    │    │
│   │  · Entry Fee & Prize Pool     │   │  · Joined Tournaments     │    │
│   │  · Live Slots (e.g. 38/48)    │   │  · Wallet Transaction Log │    │
│   │  · Match Date & Format        │   │  · Coin Top-up Request    │    │
│   │  · "Join Match" / "View Room" │   │  · In-Game ID Settings    │    │
│   └───────────────────────────────┘   └───────────────────────────┘    │
│                                                                        │
│   ┌───────────────────────────────────────────────────────────────┐    │
│   │ Developer Console (Unlocked with PIN: 7788)                   │    │
│   │  · Create Tournament (Game, Map, Slots, Fee, Prize, Schedule) │    │
│   │  · Approve/Reject Coin Recharges (UTR validation)             │    │
│   │  · Broadcast Room ID & Password to Joined Players             │    │
│   │  · Manual Coin Grant & Player Management                      │    │
│   └───────────────────────────────────────────────────────────────┘    │
└────────────────────────────────────────────────────────────────────────┘
```

#### Key Data Models:
- `Tournament`: `id`, `game` ('bgmi' | 'freefire' | 'minecraft' | 'other'), `title`, `map`, `format` ('Solo' | 'Duo' | 'Squad'), `entryFee` (coins), `prizePool` (coins), `maxSlots`, `registeredPlayers` (array of player profiles), `startTime`, `status` ('upcoming' | 'live' | 'completed'), `roomId`, `roomPassword`, `rules`.
- `UserWallet`: `balance`, `transactions` (`id`, `type`, `amount`, `description`, `timestamp`, `status`).
- `TopupRequest`: `id`, `userId`, `userName`, `amount`, `coins`, `utr`, `status` ('pending' | 'approved' | 'rejected'), `timestamp`.
- `DeveloperConfig`: `pin`, `adminUpiId`, `adminUpiName`, `coinExchangeRate` (1 INR = 1 Coin).
