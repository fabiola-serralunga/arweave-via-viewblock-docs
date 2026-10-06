## Arweave: Check Gateways from Viewblock 
Portfolio project by **Fabiola Serralunga**.

>Live availability monitor for every Arweave gateway listed by **viewblock.io** on a snapshot frozen on **2026-10-04**. The page does NOT re-query viewblock at runtime — it only verifies, in real time, whether each gateway listed that day is currently reachable and serves real Arweave data. We understand that Viewblock’s fundamental purpose today is not to provide an updated list of gateways —perhaps because AR.IO efficiently fulfills that role— but rather to serve as a blockchain explorer.


**Live demo:**
🌐 `https://arweave-via-viewblock.pages.dev` <br>
**API:**
📊🔌 `https://arweave-via-viewblock.vercel.app/api/gateways`

Stack: Vercel Serverless Functions · Cloudflare Pages · vanilla JS (no framework)

Sister project of [arweave-via-ario](https://arweave-via-ario.pages.dev). Both use the same probe transaction ID so results are directly comparable.

---
## Objetive

**The project's goal is to audit the availability of a specific list, not to mirror Viewblock forever.**

## Anything worth saving? 

**Yes, the health of the canonical gateway arweave.net.** The rest of the gateways on the list are forgettable.
First, we avoid relying on a data source that is highly likely to modify its gateway. Second, the number of live gateways provided by Viewblock is scarce, with the canonical 'arweave.net' being the only one truly available. Fourth, we understand that Viewblock’s fundamental purpose today is not to provide an updated list of gateways—perhaps because AR.IO efficiently fulfills that role—but rather to serve as a blockchain explorer.

## Viewblock

*ViewBlock was started back in 2018 with the mission to build chain-agnostic explorers and various other easy-to-use tools to help crypto adoption.* [https://viewblock.io/es/about]

As of October 6, 2026, it serves as the primary explorer for the Arweave network. The platform provides Arweave on-chain data, AO network pages, and an Arweave gateways near-stale list. Also supporting networks such as Zilliqa, THORChain, Arkeo, Kyve, and Starknet [https://viewblock.io , October 2026]. 


## The finding

Of the 498 gateways viewblock.io listed as "online" on 2026-10-04, only **~2.41% actually serve Arweave data today**. The rest fail with DNS errors (ENOTFOUND), expired TLS certs, HTTP 5xx, timeouts, or — most tellingly — return a 200 with HTML for a parked domain instead of the requested transaction.

That ~2% live rate is the reason this project exists. The frozen list is the starting point; live verification is the value.

> Credit to viewblock for keeping a public relay of gateway information. This monitor is built on top of their work — not in opposition to it.

---

## How it works

Two layers of verification per gateway, both server-side:

| Layer | What it tests | Endpoint |
|-------|---------------|----------|
| 1 — Signal | Is the gateway alive? | `GET https://<url>/info` |
| 2 — Data | Does it actually serve data? | `GET https://<url>/<PROBE_TXID>` (pin test) |

The probe TX (`1y5cosgPNeu4MXufM8W_yh7M3ZMxWrh0MUWoCqOXE4s`) is a tiny text file uploaded to Arweave whose body says `check`. **HTTP 200 + exact body match** = the gateway serves data. A gateway can have a valid `/info` JSON and still fail this probe — that's the whole point of layer 2.

### Progressive load

| Endpoint | Returns | Latency |
|----------|---------|---------|
| `/api/gateways?quick=1` | Frozen list only, `alive: null` ("checking…") | ~50 ms |
| `/api/gateways` | Full live-check + pin test results | 15-30 s |

The frontend hits `quick` first to paint immediately, then fires the full request and replaces the rows as they arrive.

---

## Columns

| # | Gateway | URL | City | Signal | Data |
|---|---------|-----|------|--------|------|

**City** is included because latency is partly a function of geographic distance from the client — it helps interpret the signal bars.

### Columns we deliberately did NOT include

- **stake / epochs / delegated stake** — those are on-chain fields of the AR.IO contract on Solana. Viewblock gateways are not registered there, so the data does not exist. We drop them rather than invent values.
- **height / network** — `height` in the frozen CSV is stale by definition; a live `/info` would give the current height, but a desynced gateway fails the pin test anyway, so height is redundant. `network` is almost always `arweave.N.1` and carries no signal.

The same principle applies to the developer's AR.IO project: 'If data cannot be accurately obtained, do not display it.

---

## Tech stack

- **Vercel serverless function** (Node 18+, `maxDuration: 60`) — pure `fetch`, no SDK dependency.
- **Cloudflare Pages** — static HTML + vanilla JS frontend (same `styles.css` as the AR.IO project, for visual consistency).
- **5 min edge cache** on the API so it doesn't re-run on every visit.
- **Session cache** on the client so a refresh within 5 min is instant.

---

## File structure

```
arweave-via-viewblock/
├── api/gateways.js              ← serverless endpoint
├── lib/viewblock_gateways.json  ← frozen snapshot, 498 gateways
├── public/
│   ├── index.html               ← progressive load UI
│   └── css/styles.css           ← shared with AR.IO sister project
├── scripts/
│   ├── build_viewblock_json.py  ← regenerates the JSON from a fresh CSV
│   ├── test_pipeline.mjs        ← smoke test, 5 gateways
│   └── test_20.mjs              ← parallel test, 20 gateways
├── package.json
├── vercel.json
└── README.md
```

---



