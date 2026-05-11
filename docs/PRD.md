# Product Requirements Document (PRD)

> **Status:** Draft
> **Last updated:** 2026-05-11
> **Author:** Carlos Stanton
> **Stakeholder:** Yourself / class project demo

---

## 1. The problem

Tidal knows your taste — its "For You" mixes and daily discoveries land often enough that you keep listening — but it won't tell you what it knows or let you have a conversation about it. You can browse curated mixes and follow recommendations, but you can't ask specific things ("play something like the last three albums I've been obsessing over, but more upbeat, and nothing I've heard in the last month"). The discovery experience is a black box: it works until it doesn't, and when it doesn't, you have no way to steer it. Heavy Tidal users — particularly music enthusiasts with large, carefully curated libraries — have no way to have a nuanced conversation about what they want to hear.

---

## 2. The user

- **Primary user:** A 20-year-old college student who is a music enthusiast. He has a large, deliberately curated Tidal library. He listens actively, not passively — he cares what's playing. He's tech-savvy and comfortable with web apps and chat interfaces. He uses his phone most, desktop sometimes.
- **Their current workflow:** Opens Tidal and browses "For You" mixes or daily discoveries. Gets broad, opaque recommendations. Has no way to refine them or understand why they were surfaced.
- **Their technical comfort:** High. Comfortable with BYOK (bring your own API key) setup if the onboarding is clear.
- **What device will they use it on:** Phone primarily, desktop secondarily.
- **The moment they open the app:** He wants to find a new artist that fits a specific vibe — not just "something good," but something specific. Tidal can't help him with that request.

---

## 3. What success looks like

- **Must-have outcome:** A user opens the app, describes what they want, and adds a song to their Tidal library that they wouldn't have found otherwise.
- **Nice-to-have outcome:** Recommendations get noticeably better after a few sessions because the app has learned from explicit feedback and in-app listening behavior.
- **A goal we want but won't measure as success in v1:** Users listen *inside* our app, not just Tidal. Every in-app listen is a signal we can use to improve their taste profile (full plays, early skips, repeats). We will encourage this in the UX, but the v1 success metric is still songs added to library — not session minutes.
- **Not a goal for v1:** Becoming a full music client. We surface playback controls only as a means to capture listening signal — not to compete with Tidal's player UX. v1 stays focused on discovery and feedback.
- **Long-term vision (v3+):** Native iOS/Mac client(s). Whichever streaming service has traction by then dictates the SDK choice — Apple MusicKit (Swift) if Apple Music dominates app usage, or Tidal's native SDKs if Tidal does. The framing is *augmenting* the user's chosen streaming service, not replacing it. v1 proves the discovery concept on the web against a single service; v2 broadens to multiple premium services + a free tier; v3+ brings native depth. This vision should shape v1 decisions: don't paint yourself into a corner that makes a future port painful.
- **Free tier (v2):** A YouTube-backed free tier. User signs in with Google, the app reads their existing YouTube data (playlists, liked videos, music-channel subscriptions) for cold-start taste signal, and creates a private playlist that becomes their persisted library. Same conceptual model as Tidal — the library lives in the music service, the app reads/writes against it — different API surface. Lets the product have a real free tier without scraping or ToS violations.
- **Apple Music support (v2):** The most-established premium music service, with a large catalog, mature web SDK (MusicKit JS), and target-persona overlap with music enthusiasts. Same conceptual shape as Tidal — catalog metadata, library writes, in-browser playback. Deferred to v2 because the Apple Developer account costs $99/yr; pushing this to v2 lets v1 prove the discovery loop at near-zero cost before committing the fee. Planned as a parallel premium tier alongside Tidal.
- **Kill condition (v2):** Tidal restricts the Web API or Player SDK to the point where the core features no longer work.

---

## 4. Core user stories

1. **[Must]** As a music enthusiast, I want to describe what I'm looking for in natural language so that I can find a new artist or song that fits a specific mood or context that Tidal can't surface on its own.

2. **[Must]** As a user, I want the app to silently learn my taste from my library contents and my explicit feedback (liked, disliked, added to library) so that recommendations improve over time without me having to do anything extra.

3. **[Must]** As a user, I want to see recommendations as cards with album artwork and buttons to play, add to library, like, or dislike so that acting on a recommendation is frictionless.

4. **[Should]** As a user, I want to see my recommendation history with my explicit ratings so that I can revisit past suggestions.

5. **[Should]** As a user, I want to build a playlist from recommendations and add it to my Tidal library so that I can listen to it outside the app.

6. **[Could]** As a user, I want the app to build me a taste avatar that evolves visually with my listening preferences so that I can see how my taste is changing over time.

7. **[Won't — v1]** Apple Music support, Spotify support, YouTube support, native iOS/Mac app, monetization, paid AI tier.

---

## 5. Out of scope

- **Native SDKs (Apple MusicKit Swift, Tidal native SDKs)** — v3+; v1 is web-only
- **Play count / listening history from the streaming service** — most streaming APIs do not expose granular play counts to third parties; we build our own signal from in-app behavior
- **Spotify integration** — not viable. As of Feb 2026, Spotify deprecated the endpoints a discovery app needs (audio features, audio analysis, related artists, recommendations, editorial playlists). New apps are also capped at 5 manually allowlisted users in "Development Mode," and "Extended Quota Mode" effectively requires an existing 250k+ MAU business (>95% of applications rejected per Spotify's own blog). This is a closed door, not a v2 consideration.
- **Apple Music integration** — out of scope for v1. Viable for v2 (see §3); deferring avoids the $99/yr Apple Developer account cost during v1 prototyping and keeps v1 focused on a single streaming service surface.
- **YouTube integration** — out of scope for v1. The intended v2 shape is a YouTube-playlist-backed free tier (see §3), not YouTube as a generic catalog source. Holding it for v2 keeps v1 focused on the Tidal discovery loop.
- **Native iOS or Mac app** — v2; v1 is a web app
- **Paid/managed AI tier** — v2; v1 is BYOK only
- **Social or collaborative features** — not in v1
- **Siri or voice input** — not in v1

---

## 6. Technical shape

- **Type of app:** Full-stack web app. Frontend served from Cloudflare Pages, backend logic in Cloudflare Workers.
- **Does it need to store data?** Yes — user taste profile, recommendation history, explicit feedback (liked/disliked/added), in-app listening session events.
- **Does it need authentication?** Yes. Users log in so their taste profile and history persist across sessions and devices. Auth is required (not optional) because the taste profile is the core value of the app.
- **Does it need to call external services?** Yes — TIDAL Web API + Player SDK (catalog + playback + library), and the user's own LLM API key (Google AI Studio / Gemini as the primary recommended option, others supported).
- **Who pays for hosting?** Developer (Carlos) pays for Cloudflare. Users pay for their own LLM API keys. TIDAL developer access is free; the Player SDK requires an active Tidal subscription on the signed-in account for full-track playback during testing — covered by the 30-day free trial for v1 development.

### Proposed Cloudflare stack

| Need | CF Product | Why |
|---|---|---|
| Hosting the web UI | Pages | Free static + serverless hosting, deploys from git |
| Backend API | Workers | Serverless functions for taste profile, recommendations, session tracking |
| Structured data | D1 | SQL database for users, taste profiles, recommendation history, feedback events |
| Cross-device sync | R2 | Store library snapshot JSON for fast cold-start profile building |
| AI call proxying | AI Gateway | Proxies the user's own API key — adds logging and rate limiting for free |
| Session / auth tokens | KV | Fast global key-value store for auth tokens |

### TIDAL Web API & Player SDK — known limitations

The platform is newer than Spotify's or Apple's developer offerings; expect to discover undocumented behavior during Phase 1. Provisional list to verify empirically:

- Play counts and granular listening history are unlikely to be exposed to third-party apps (consistent with the broader streaming-API landscape). Taste profile must be built from library contents + in-app session tracking + explicit feedback.
- The Player SDK requires an active Tidal subscription on the signed-in account for full-track playback; catalog and metadata endpoints work without one.
- Skip detection will likely need to be inferred (track change before completion), not a native event. Expect ~1–2 second precision.
- Library pagination behavior, rate limits, and 429 backoff semantics to be characterized in Phase 1.
- OAuth 2.1 (authorization code + PKCE) — refresh-token handling on a Workers backend needs care; KV is the right place to keep tokens, not the client.
- You can only track listening behavior that happens inside your app.

---

## 7. Risks and unknowns

- **Biggest risk:** TIDAL's Web API and Player SDK are newer and less battle-tested than mature alternatives. Expect to discover undocumented behavior mid-build. Allocate buffer time for API surprises.
- **Second biggest risk:** Understanding the code well enough to answer unscripted questions at demo. Build simply, use the rubber-duck-quiz skill before every commit, and favor code you can explain over code you can't.
- **Cold-start quality:** Library contents alone may not produce good enough recommendations to hook a new user before they've built up in-app feedback history. This needs to be tested early — week 2.
- **Things I don't know how to do yet:** TIDAL OAuth 2.1 + PKCE flow on a Workers backend, Cloudflare Workers + D1 data modeling, LLM prompt engineering for music recommendations.
- **Things I'm assuming but haven't verified:** That library analysis (artist diversity, genre spread, era distribution) produces meaningful taste signals without play counts; that Tidal's catalog covers enough of the developer's curated library to make cold-start testing realistic.

---

## 8. Milestones

- **Week 2 end:** TIDAL OAuth working, user library loaded and stored, basic chat UI deployed to Cloudflare Pages and publicly accessible. Cold-start recommendations generated from library contents and displayed as cards. This is the riskiest week — if TIDAL auth or library access hits a wall, surface it here, not in week 4.
- **Week 3 end:** LLM connected with user's own API key (Google AI Studio onboarding flow complete), recommendations responding to natural language mood/context input, like/dislike/add-to-library buttons functional and writing feedback to D1.
- **Week 4 demo:** Taste profile visibly improving recommendations based on accumulated feedback. App is publicly deployed, demo-ready, and the developer can answer unscripted questions about how every part works.
