# AJS TRADING PRO — NIFTY Box-1 Test Report

## Automated checks passed
- `npm run check`: `server.js` and `public/app.js` syntax passed.
- HTML parsing: 101 IDs checked, no duplicate IDs.
- Instrument configuration: NIFTY + BANK NIFTY + all 11 selected stocks configured.
- Box-1 required elements present: CALL score, PUT score, side-labelled score-based confidence, advance/decline counts, and breakout watch.
- Advance/decline now compares live LTP against the latest completed daily candle (previous session close), not the candle before that.
- Breakout watch has blinking CSS states for watch/trigger.
- Live NIFTY 5-minute candle updates are passed into breakout-watch logic so the next candle's high/low can trigger before that candle closes, after the initial breakout/breakdown candle has closed.
- ZIP archive integrity passed.
- Local smoke test: `/` and `/app.js` returned HTTP 200; `/api/status` returned 13 configured instrument entries (2 indices, 11 stocks) with no token configured.

## Not verified
- Live Upstox login/token permissions, live ticks, option chain, actual PCR/OI, and browser behavior during market hours were not verified because the smoke test intentionally ran without a token.
- Direction confidence is a score-based heuristic, not a validated probability or guarantee.
- Paper trading only; real order execution remains disabled.

## Start
Extract the ZIP, configure your Upstox token in the app's local `.env` file or settings as documented, and run `START_AJS_TRADING.bat`. Test the breakout trigger in paper mode during market hours before relying on it.


## OI FLOW PRO implementation update
- Added `/api/oi-flow` endpoint with active NSE futures discovery from Upstox instrument master and batched quote retrieval.
- Added standalone OI Flow dashboard section with signal filter/search and price/OI metrics.
- Existing CALL/PUT scoring functions and parameters were not modified.
- `npm run check`: PASS (Node syntax checks for server.js and public/app.js).
- OI classifier unit checks: PASS for Long Build-up, Short Build-up, Short Covering, and Long Unwinding.
- Live Upstox market verification: NOT PERFORMED in this build step; requires a valid token, API access and an active market session.

## Smoothness / trigger safety correction (2026-10-20)
- Fixed a confirmation edge case: the follow-up trigger candle is now selected only from the completed-candle list. An in-progress 5-minute candle can no longer trigger the celebration early.
- Removed the unused OI Flow polling call from the main dashboard refresh because the OI Flow / Futures Positioning panel is removed from the UI.
- `npm run check`: PASS.
- Trigger close-only guard check: PASS.
- Local smoke test: `/` HTTP 200, `/app.js` HTTP 200, `/api/status` returned 13 instruments.
- Live market data test: NOT VERIFIED; `/api/candles/NIFTY/5m` and `/api/option-chain/NIFTY` returned `fetch failed` in this environment. Re-test with a valid, current Upstox API connection during market hours.


## Additional review and correction (2026-10-20)
- Fixed stale breakout selection: if a breakout's available B1 retest/continuation fails, the scanner now continues looking for a later valid breakout instead of repeatedly showing the same invalid setup.
- The candidate scan also skips setups whose available B2 candle failed confirmation, allowing a later valid setup to be considered.
- `npm run check`: PASS after patch.
- Local no-token smoke test: `/` HTTP 200, `/app.js` HTTP 200, `/api/status` HTTP 200; 13 instruments configured; `paperTrading=true`, `realOrders=false`.
- `/api/candles/NIFTY/5m` correctly returned HTTP 401 with a clear missing-token error in the no-token test.
- Live candles, option premiums, browser behavior during market hours, and live Upstox permissions remain NOT VERIFIED.
- Security: the uploaded archive contained a non-empty `.env` access-token value. This delivery excludes `.env`; revoke/regenerate that Upstox token before continuing if the uploaded archive was shared or exposed.
