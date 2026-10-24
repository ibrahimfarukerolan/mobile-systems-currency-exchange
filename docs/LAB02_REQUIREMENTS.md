# Lab 02 — Requirements and Network Scenarios

**Project:** Mobile Systems — Currency Exchange  
**Status:** Proposed requirements for implementation and testing  
**Architecture:** Android (Kotlin) · Spring Boot (Java) · PostgreSQL · National Bank of Poland (NBP) API

> **Scope:** Currency buying and selling use a **virtual wallet and simulated funds**. No real money is transferred. The backend is the authority for account balances and completed transactions.

## 1. Project Overview

The mobile application lets registered users view NBP exchange rates, calculate currency conversions, check virtual wallet balances, simulate currency buying and selling, and review transaction history. The Android app communicates with a Spring Boot backend over HTTPS. PostgreSQL stores user accounts, wallet balances, transactions, and server-side rate snapshots. NBP is the external source of exchange-rate data.

**Main network rule:** Reading cached information is allowed offline, but changing server data is not. A lost response to a submitted transaction is an **unknown outcome**, not proof that the transaction failed.

## 2. Functional Requirements

Each requirement includes a result that can be checked during testing.

### FR-01 — User registration
**Requirement:** A user can register with a valid email and password while online.  
**Acceptance test:** Valid input creates one account; invalid input or a duplicate email returns a clear error and creates no account.

### FR-02 — User login and logout
**Requirement:** A registered user can log in and log out. Protected wallet and transaction endpoints require authentication.  
**Acceptance test:** Valid credentials allow access; invalid credentials are rejected; after logout the app no longer exposes authenticated screens without a new login.

### FR-03 — View exchange rates
**Requirement:** A user can view supported currency rates obtained from NBP, with the rate date and the time the app fetched them.  
**Acceptance test:** An online request displays rates with their timestamps; if the source is unavailable, the app displays an error or clearly marked cached data.

### FR-04 — Currency conversion
**Requirement:** A user can enter a positive amount and select supported currencies to see a calculated conversion.  
**Acceptance test:** The result matches the selected rate and rounding rule; zero, negative, and invalid input is rejected. Conversion is informational and does not change a wallet.

### FR-05 — View virtual wallet
**Requirement:** An authenticated user can view confirmed balances for supported currencies.  
**Acceptance test:** The displayed online balance matches the backend; any offline snapshot is labeled as not verified/current.

### FR-06 — Simulated buy and sell
**Requirement:** An authenticated user can submit a simulated exchange using a server-provided quote, and the backend validates funds and updates both currency balances in one database transaction.  
**Acceptance test:** A valid request changes the balances once and creates one transaction record; insufficient funds or an expired quote causes rejection without a balance change.

### FR-07 — Transaction history
**Requirement:** An authenticated user can view their confirmed exchange transactions and their status.  
**Acceptance test:** Each confirmed transaction appears once with its ID, currencies, amounts, rate, timestamp, and status.

### FR-08 — Offline cached display
**Requirement:** The app can display the last saved exchange rates and previously confirmed wallet/history snapshots when disconnected.  
**Acceptance test:** Offline screens show cached values and their last-sync times with a visible **Cached / May be outdated** label; no new server data is claimed.

### FR-09 — Refresh and reconciliation
**Requirement:** After reconnection, the app can refresh rates and query the backend for any transaction whose response was lost.  
**Acceptance test:** Refreshed values replace cached values; an unresolved transaction is checked by request ID before a user can submit it again.

## 3. Non-Functional Requirements

### NFR-01 — Transport security
All mobile-to-backend traffic must use HTTPS in deployed environments. **Test:** HTTP access is rejected or redirected; no credentials are transmitted over plaintext HTTP.

### NFR-02 — Response time
Under a defined lab test setup with a reachable backend, at least 95% of cached/server-available rate reads should finish within **3 seconds**. **Test:** Record the response times of 100 requests; at least 95 meet the target. *This is a proposed project target, not a value provided by the lecturer.*

### NFR-03 — Network resilience
The app must not crash when Wi-Fi/mobile data is disconnected, slow, or restored. **Test:** Repeat read and write scenarios with airplane mode and network throttling; the app displays an appropriate state.

### NFR-04 — Data freshness
Every rate shown must include its NBP effective date and local retrieval time; cached views must be labeled. **Test:** Disconnect after a successful fetch and confirm the original timestamps remain visible.

### NFR-05 — Idempotent transactions
Each buy/sell request must have a unique client-generated request ID. Repeating an identical request ID must not execute the transaction twice. **Test:** Submit the same ID twice and verify exactly one transaction and one set of balance changes. Reuse of an ID with different payload is rejected.

### NFR-06 — Authorization and privacy
Users may access only their own wallet and transaction history. **Test:** An authenticated user requesting another user's data receives an authorization error.

### NFR-07 — Consistency under concurrency
Wallet updates must be atomic and protected against competing requests. **Test:** Run two simultaneous requests that together exceed the available funds; the final balances must never be negative and only affordable requests succeed.

### NFR-08 — Clear feedback
Every network-dependent screen must show a loading, success, error, offline, or pending-confirmation state. **Test:** Simulate each network scenario and verify the matching state is visible.

## 4. Read vs. Authoritative Actions

**Read-only / informational:** Exchange rates, currency calculations, wallet viewing, and transaction-history viewing. Previously saved values may be shown offline **only with a cache warning and timestamp**.

**Server-authoritative / data-changing:** Registration and simulated buy/sell. These require an online backend response or later status reconciliation. A timeout or missing response must **not** be interpreted as a rejected transaction.

**Offline decision:** **Block new writes; do not queue them.** If a write was already sent before disconnection, retain its request ID and mark the result **Pending confirmation** until the backend is checked.

## 5. Network Scenario Matrix

| ID | Network event / operation | User-visible state | Server state | Expected test result |
|---|---|---|---|---|
| NS-01 | Connection lost **before** buy/sell submission | Offline warning; submit disabled | No request received; balances unchanged | Disconnect first, attempt submit: zero transactions created |
| NS-02 | Connection lost **after** buy/sell submission | **Pending confirmation**; no success/failure claim | May have committed or rejected the request | Drop response; reconnect and query by request ID; final UI matches server record |
| NS-03 | Read request times out | Timeout with Retry; cached rate if available | No wallet change | Delay rate response beyond client timeout; no crash and no invented fresh rate |
| NS-04 | Write request times out | **Pending confirmation**; check status action | Write may have committed | Delay transaction response; query request ID before any retry; no duplicate debit |
| NS-05 | Response arrives late | Loading/pending until resolved; no conflicting messages | Original request processed once | Deliver late response; accept only if it matches the outstanding request ID |
| NS-06 | User retries a rate read | Retry indicator; then new data or error | No wallet change | Repeat GET safely; UI shows the newest valid response |
| NS-07 | User retries an uncertain buy/sell | Pending status while reconciling | Original transaction may exist | Same request ID returns original result, never creates a second trade |
| NS-08 | Cached rates displayed offline | **Cached / May be outdated** plus effective and fetch dates | No new server request or change | Disconnect and open rates: saved values appear with warning |
| NS-09 | Connection returns | Reconnecting, then refreshed rates and resolved statuses | Backend provides current records | Restore network; rates refresh and pending transaction reconciles |
| NS-10 | Two competing buys use the same funds | One or both requests rejected if funds insufficient | Updates serialized/atomic; no negative balance | Send concurrent requests exceeding funds together; balances stay valid |
| NS-11 | NBP unavailable but backend reachable | Source unavailable; cached rates clearly marked | Backend retains last known rate snapshot | Simulate NBP failure; no stale rate is presented as live |
| NS-12 | Two requests for rates return out of order | Latest requested rate selection remains on screen | Read-only; no balance change | Delay first response; confirm older response cannot overwrite newer selection |

## 6. Test Setup and Evidence

Use an Android emulator or test device, a running Spring Boot service, PostgreSQL, and a controlled NBP response or mock data for repeatable tests. Simulate network loss using airplane mode or emulator controls; use server-side delays to reproduce timeouts and late responses.

For each scenario, capture: **scenario ID, steps, expected UI, observed UI, expected backend state, observed backend state, pass/fail**. For data-changing tests, inspect transaction IDs and database balances before and after execution.


