## **1\. Revision Overview**

This document outlines critical updates to the initial StockForge architecture to ensure compliance with hackathon bounty rules and to guarantee the functional integrity of our core features. The updates focus on securing our eligibility for the PreStocks track and fixing the underlying mechanics of our ETF baskets before finalizing the codebase for submission.

## **2\. Securing the $10K PreStocks Bounty (Tessera Removal)**

### **The Problem**

&nbsp;PRD specified the integration of both PreStocks and Tessera (T-tokens) to supply pre-IPO underlying assets for the options desk and custom baskets. However, the official rules for the $10,000 PreStocks bounty explicitly state: *"Note: projects that integrate any non-PreStocks pre-IPO tokens will be ineligible for this bounty."* By keeping Tessera in the codebase, we trigger automatic disqualification from the $10,000 PreStocks track and compromise our positioning for the $100K Main Track pool.

### **The Solution: 100% PreStocks Architecture**

* **Action:** Completely strip Tessera and all T-tokens (OpenAI, Kalshi, SpaceX) from the project scope.  
* **Implementation:** We will source all pre-IPO asset data exclusively through the PreStocks API (\[https://prestocks.com/api/prestocks\](https://prestocks.com/api/prestocks)). All underlying assets for pre-IPO options contracts and ETF baskets will utilize official PreStocks SPL tokens.  
* **Strategic Impact:** This necessary sacrifice forfeits the $6K Tessera track but fully secures our eligibility for the $10K PreStocks bounty (1st: $5k, 2nd: $3k, 3rd: $2k). Given that PreStocks offers the necessary pre-IPO exposures, the core user flow remains identical while aligning perfectly with the primary sponsor's compliance rules.

## **3\. Critical Fix 2: Decoupling ETF Baskets from Meteora DBC**

### **The Problem**

The previous architecture mandated that every time a user creates a new ETF basket (e.g., an "AI Titans" index), the smart contract mints a composite SPL token and simultaneously launches a Meteora Dynamic Bonding Curve (DBC) pool. This breaks the fundamental mechanics of an asset-backed ETF. Meteora DBCs are Automated Market Makers (AMMs) designed to exchange a quote currency (like USDC) for a newly launched token. If users purchase the basket token via the DBC, the pool accumulates USDC—not the underlying NVDA and MSFT stock tokens. Consequently, when a user attempts to burn their basket token, the vault will fail to return the underlying assets because it never held them.

### **The Solution: Dedicated 1:1 Vaults \+ Distinct DBC Launch**

* **Action:** Separate the ETF Basket logic from the Meteora DBC liquidity logic.  
* **Basket Vault Implementation:** Build a standard Anchor vault for the ETF. The vault will enforce a strict 1:1 pro-rata ratio. To mint a BasketSPL token, users must deposit the exact underlying stock tokens into the vault (e.g., depositing 2 NVDA and 1 MSFT). To redeem, the user burns the BasketSPL token, and the vault executes a Cross-Program Invocation (CPI) to transfer the underlying stock tokens back to their wallet.  
* **Meteora DBC Bounty Alignment:** To qualify for the $5K Meteora DBC track without breaking the ETF vault, we will utilize the DBC strictly for a single flagship token launch (e.g., launching a "StockForge Governance Token" or a specific equity-paired pool). We will use Meteora's TypeScript SDK to apply a linear pricing curve and custom fee tier, directly addressing Meteora’s prompt for "launch mechanics tuned for equity-like assets."

## **4\. Minor Fix 3: Manual Price Feed Fallback for Pre-IPO Stocks**

* **The Problem:** Pyth Network lacks on-chain price accounts for unlisted pre-IPO assets, which will cause smart contract failures during settlement.  
* **The Solution:** We will implement a dual-routing system. For public stocks, the contract reads standard Pyth on-chain accounts. For pre-IPO assets, our Next.js frontend will fetch the settlement price from the live PreStocks API, pass it to the contract as an argument, and verify it on-chain using an admin signature.

## **5\. Minor Fix 4: Permissionless Option Settlement (The "Crank")**

* **The Problem:** Solana smart contracts cannot automatically execute on a timer. If an option expires, funds remain locked unless explicitly settled.  
* **The Solution:** We will make the settle\_option instruction permissionless so any wallet can call it post-expiration. We will allocate a negligible portion of the locked escrow to the caller as an incentive, ensuring network participants automatically trigger our expired contract settlements.