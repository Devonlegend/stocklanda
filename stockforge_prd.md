# StockForge — Product Requirements Document

| Metadata | Details |
| :--- | :--- |
| **Hackathon** | Stocklana |
| **Deadline** | September 25, 2026 |
| **Version** | 0.1 |
| **Status** | Draft |
| **Bounty Targets** | Main Track ($100K pool) · PreStocks ($10K) · Meteora DBC ($5K) · Tessera ($6K) · Pyth Network (3mo Pro) |

---

## 1. Problem Statement

Tokenized stocks trade on Solana 24/7, but holders are currently limited to a single action: **holding and selling**. 

There is currently no mechanism to:
* Hedge downside risk with options contracts.
* Generate passive yield by writing covered calls.
* Pool assets into a shared, composable index or custom basket for group investing.

The result is **dead capital**: users hold tokenized equity without access to the foundational financial derivatives and structured products that make equity functional and liquid.

---

## 2. Proposed Solution

StockForge delivers two interconnected primitives within a unified interface:

### 2.1 P2P Options Desk
* **Mechanics:** Writers deposit collateral to list fully backed put or call options on any tokenized stock at a fixed strike and expiration timestamp. Buyers purchase contracts by paying an upfront premium.
* **Settlement:** At expiration, Pyth Network oracle feeds settle the contract on-chain without intermediaries, market open/close restrictions, or KYC requirements.
* **Core Scenario:** Purchase downside protection on a pre-IPO SpaceX position at 2:00 AM on a Sunday.

### 2.2 Custom Basket ETFs
* **Mechanics:** Any user can define an asset basket (e.g., *"AI Titans"*: 40% NVDA, 35% MSFT, 25% AAPL tokens) and mint a single SPL token representing the composite allocation.
* **Liquidity & NAV:** Other market participants can subscribe proportionally. Net Asset Value (NAV) is calculated in real time via Pyth price feeds.
* **Redemption:** Token holders can burn the basket token at any time to redeem their pro-rata share of underlying tokens.
* **Composability:** Basket tokens serve as valid underlyings for the options desk, enabling single-trade downside hedging across an entire thematic portfolio.

---

## 3. Core User Flows

### Flow A: Options ("Panic Put")
```text
[Writer] ---> Locks USDC collateral & sets strike/expiry
               ↓
[Buyer]  ---> Pays premium; funds locked in escrow contract
               ↓
[Oracle] ---> Pyth price feed queried at expiry timestamp
               ↓
[Payout] ---> ITM: Buyer receives payout
         ---> OTM: Writer reclaims collateral + keeps premium
```

1. **Listing:** Writer locks USDC collateral, defines strike price, and selects expiry timestamp.
2. **Escrow:** Buyer pays premium; contract goes live on-chain in program escrow.
3. **Settlement:** Pyth price feed settles the position at expiry timestamp.
4. **Resolution:**
   * **In-The-Money (ITM):** Buyer receives payout from locked collateral.
   * **Out-of-The-Money (OTM):** Writer reclaims collateral and retains premium.

---

### Flow B: Baskets ("Group ETF")
```text
[Creator] ---> Composes basket & sets weights
               ↓
[Program] ---> Mints basket SPL token & creates Meteora DBC pool
               ↓
[Users]   ---> Purchase basket token; mint proportional shares
               ↓
[Redeem]  ---> Burn basket token to retrieve underlying assets
```

1. **Composition:** Creator selects supported tokenized stocks, assigns weights, and names the basket.
2. **Minting & Liquidity:** Program mints a single basket SPL token and initializes a Meteora Dynamic Bonding Curve (DBC) pool.
3. **Participation:** Community members purchase basket tokens; shares are allocated proportionally.
4. **Redemption:** Holders burn the basket token at any time to receive their share of the underlying tokens.

---

## 4. Feature Scope

| Feature | v1 Status | Notes |
| :--- | :---: | :--- |
| **Cash-secured put options** | ✅ In | USDC-collateralized, Pyth-settled |
| **Covered call options** | ✅ In | Writer locks underlying stock tokens rather than USDC |
| **Basket mint & redeem** | ✅ In | Create basket, share link, deposit liquidity, burn to exit |
| **Live basket NAV display** | ✅ In | Pyth price feed per holding, calculated weighted sum |
| **PreStocks + Tessera underlyings** | ✅ In | OpenAI, SpaceX, and Kalshi T-tokens supported |
| **Options on basket tokens** | ✅ In | Primary differentiator: hedge an entire basket in one trade |
| **Auto-rebalancing** | ⏳ Post-v1 | Manual redeem and re-enter flow for initial release |
| **Options spreads / straddles** | ⏳ Post-v1 | Single-leg options only at launch |
| **Secondary options market** | ⏳ Post-v1 | Hold-to-expiry execution only in v1 |

---

## 5. Technical Architecture

| Layer | Technology | Purpose |
| :--- | :--- | :--- |
| **Core Program** | Anchor (Rust) | On-chain escrow, minting, and settlement logic |
| **Price & NAV Settlement** | Pyth Network | Low-latency on-chain price feeds |
| **Basket Token Pools** | Meteora DBC | Dynamic bonding curve pool creation for basket SPL tokens |
| **Pre-IPO Token Data** | PreStocks API | Metadata and market data for pre-IPO assets |
| **Pre-IPO Token Assets** | Tessera T-tokens | Underlyings (OpenAI, Kalshi, SpaceX) |
| **Frontend Interface** | Next.js + Tailwind CSS | Dashboard, basket composer, and options desk |
| **Wallet Integration** | Solana Wallet Adapter | Phantom, Solflare, and Backpack connection |
| **Deployment Target** | Solana Devnet → Mainnet | Devnet testing leading to mainnet deployment |

---

## 6. Build Timeline (Hackathon Sprint)

| Phase | Focus | Key Deliverables |
| :--- | :--- | :--- |
| **Day 1 (Wed)** | Program Skeleton | Anchor accounts, Pyth feed reading on Devnet, wallet connect |
| **Day 2 (Thu)** | Options Logic | `write_option`, `buy_option`, `settle` instructions + listing/trading UI |
| **Day 3 (Fri)** | Baskets + DBC | Basket mint/redeem, Meteora DBC integration, live NAV display widget |
| **Day 4 (Sat)** | Polish & Submit | Tessera & PreStocks assets wired, demo recording, submission pitch |

---

## 7. Prize & Bounty Alignment

| Bounty Track | Qualification Strategy | Target Prize |
| :--- | :--- | :--- |
| **Main Track** | Combines Trading (options desk), Investing (baskets), and Collateral wedges | $100K pool |
| **PreStocks** | Direct support for derivatives and structured products on pre-IPO tokens | Up to $10K |
| **Meteora DBC** | Implementation of novel dynamic bonding curve configurations for basket launches | Up to $5K |
| **Tessera** | Use of OpenAI and Kalshi T-tokens as option and basket underlyings | Share of $6K |
| **Pyth Network** | Mission-critical role in both basket NAV indexing and automated option settlement | 3 months Pro access |

---

## 8. Why Solana?

* **24/7 Derivative Settlement:** Traditional equity options cease trading after regular market hours. StockForge leverages Solana's round-the-clock block production to offer continuous hedging and execution.
* **Sub-Cent Transaction Fees:** Micro-premiums and small-dollar fractional basket purchases remain economically viable for retail traders.
* **Deep Composability:** Basket tokens are standard SPL tokens that seamlessly compose with broader DeFi protocols, allowing them to act as lending collateral, LP inventory, or option underlyings.

---

## 9. Key Differentiator

> **Options on basket tokens.**  
> Currently, there is no on-chain mechanism to write a put option on a customized bundle of pre-IPO equities. StockForge enables a user to hedge an entire themed portfolio—such as SpaceX + OpenAI + NVDA—within a single on-chain transaction.