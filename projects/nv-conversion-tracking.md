# NV Health — conversion tracking (GTM + stape server-side)

Authoritative, distilled from the expert's 4 Loom walkthroughs (2026-07-05) + the live GTM containers.
Drives the syd.nvhealth.com.au ad-landing conversion wiring. See [[nv-health-website]].

## Architecture (server-side via stape)
- **Web container** `GTM-M9MD7MPR` (loads on every NV site incl. syd). Has a GA4 **Google Tag** on All Pages
  whose `server_container_url` = **https://server.nvhealth.com.au** (+ `send_page_view=true`). THAT parameter is
  what makes the whole thing server-side — it reroutes ALL GA4 hits to the stape server instead of Google.
- **Server container** `GTM-KFWJ9BDZ`, tagging URL **server.nvhealth.com.au** (stape). Has a GA4 **client**
  (claims inbound requests) + tags that fire on a **Custom Event where Client Name contains `GA4`**:
  - **Google Ads - Consult Booking** (Conversion Tracking, Ads ID **AW-11431360945**) — fires on event **`generate_lead`**.
  - **Meta** (Facebook Conversion API by stape) — fires on `generate_lead`. Dedup via a GTM-side "unique event id"
    variable + fbp/fbc cookies (NOT from the site's dataLayer).
  - GA4 forward, Google Ads conversion-linker, remarketing.

## THE key fact
The server-side conversion keys on **ONE thing: a GA4 event named exactly `generate_lead`** arriving via the GA4
client. Rename it → conversion silently stops. **It does not care which site sent it** — expert: *"I don't need
to do anything in the server side? No... everything is working there."*

## Plan for syd (the new site)
1. **Site fires `dataLayer.push({event:'generate_lead'})` on booking success.** ✅ DONE + verified (BookingForm
   `confirmBooking` → `pushEvent({name:'generate_lead'})`; also fires internal `booking_confirmed`, `details_submitted`,
   `slot_selected` for GA4 funnel). Env: `NEXT_PUBLIC_GTM_ID=GTM-M9MD7MPR`, `NEXT_PUBLIC_GTM_SERVER_URL=https://server.nvhealth.com.au`
   (set in .env.local; **STILL TODO: add both to Vercel env for prod**).
2. **Web container: add a Custom Event trigger for `generate_lead`** + attach to the existing GA4 `generate_lead`
   event tag. Server container UNCHANGED. (Pending — walk Eli through it.)
3. **Validate**: GTM Preview (both containers) + fire a test booking → confirm the conversion reaches Google Ads.

## GOTCHAS (expert-flagged)
- **stape free tier caps ~10k requests/mo → tracking STOPS when hit.** A high-traffic Ads landing page burns it
  faster. CONFIRM Eli is on the paid (~$20/mo) stape plan before scaling ad spend. (Flagged to Eli 2026-07-05.)
- **Google Ads has NO dedup** → the web container's client-side Google Ads tags are deliberately PAUSED; Ads fires
  server-side ONLY. Do NOT unpause them.
- **Duplicate GA4** (hardcoded in site head AND in GTM) breaks server forwarding — "most popular issue." Our site
  loads GA4 only via GTM, so fine.
- **Verify the live event-name string** in the server Ads conversion trigger — was `consult_booking` in the original
  build, now `generate_lead` (confirmed from screenshots). Match whatever's actually live.
- Enhanced conversions (hashed email/phone) NOT set up — a gap, not a dependency. Bare `generate_lead` registers
  the Ads conversion (value defaults to the conversion action's default; pass value/currency later if wanted).
- No Consent Mode gating exists in this build.
