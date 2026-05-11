# Product Requirements Document (PRD)

> **Status:** Draft
> **Last updated:** 2026-05-07
> **Author:** Carlos Stanton
> **Stakeholder:** Yourself / class project demo

---

## 1. The problem

Apple Music knows your taste — it must, because its recommendations usually work — but it won't tell you what it knows or let you have a conversation about it. You can ask Siri broad things ("play something I'll like") but you can't ask specific things ("play something like the last three albums I've been obsessing over, but more upbeat, and nothing I've heard in the last month"). When asked directly, Siri responds with a list of playback commands instead of an answer. The discovery experience is a black box: it works until it doesn't, and when it doesn't, you have no way to steer it. Heavy Apple Music users — particularly music enthusiasts with large, carefully curated libraries — have no way to have a nuanced conversation about what they want to hear.

---

## 2. The user

- **Primary user:** A 20-year-old college student who is a music enthusiast. He has a large, deliberately curated Apple Music library. He listens actively, not passively — he cares what's playing. He's tech-savvy and comfortable with web apps and chat interfaces. He uses his phone most, desktop sometimes.
- **Their current workflow:** Opens Apple Music or asks Siri. Gets broad, opaque recommendations. Has no way to refine them or understand why they were surfaced.
- **Their technical comfort:** High. Comfortable with BYOK (bring your own API key) setup if the onboarding is clear.
- **What device will they use it on:** Phone primarily, desktop secondarily.
- **The moment they open the app:** He wants to find a new artist that fits a specific vibe — not just "something good," but something specific. Apple Music can't help him with that request.

---

## 3. What success looks like

- **Must-have outcome:** A user opens the app, describes what they want, and adds a song to their Apple Music library that they wouldn't have found otherwise.
- **Nice-to-have outcome:** Recommendations get noticeably better after a few sessions because the app has learned from explicit feedback and in-app listening behavior.
- **A goal we want but won't measure as success in v1:** Users listen *inside* our app, not just Apple Music. Every in-app listen is a signal we can use to improve their taste profile (full plays, early skips, repeats). We will encourage this in the UX, but the v1 success metric is still songs added to library — not session minutes.
- **Not a goal for v1:** Becoming a full music client. We surface playback controls only as a means to capture listening signal — not to compete with Apple Music's player UX. v1 stays focused on discovery and feedback.
- **Long-term vision (v2+):** A native iOS/Mac app built on the Swift MusicKit SDK that serves as a **taste-aware companion client** to Apple Music — full playback, queue management, and the data access (play counts, real listening history) the web API doesn't expose, with our chat-driven discovery layer on top. The framing is *augmenting* Apple Music, not replacing it; Apple's MusicKit program exists to enable exactly this kind of third-party client. v1 proves the discovery concept on the web; v2 brings it to native with deeper integration. This vision should shape v1 decisions: don't paint yourself into a corner that makes the v2 port painful.
- **Kill condition (v2):** Apple restricts the MusicKit API to the point where the core features no longer work.

---

## 4. Core user stories

1. **[Must]** As a music enthusiast, I want to describe what I'm looking for in natural language so that I can find a new artist or song that fits a specific mood or context that Apple Music can't surface on its own.

2. **[Must]** As a user, I want the app to silently learn my taste from my library contents and my explicit feedback (liked, disliked, added to library) so that recommendations improve over time without me having to do anything extra.

3. **[Must]** As a user, I want to see recommendations as cards with album artwork and buttons to play, add to library, like, or dislike so that acting on a recommendation is frictionless.

4. **[Should]** As a user, I want to see my recommendation history with my explicit ratings so that I can revisit past suggestions.

5. **[Should]** As a user, I want to build a playlist from recommendations and add it to my Apple Music library so that I can listen to it outside the app.

6. **[Could]** As a user, I want the app to build me a taste avatar that evolves visually with my listening preferences so that I can see how my taste is changing over time.

7. **[Won't — v1]** Spotify support, YouTube support, native iOS/Mac app, monetization, paid AI tier.

---

## 5. Out of scope

- **MusicKit native SDK (Swift)** — requires learning Swift; web API is sufficient for v1
- **Play count / listening history from Apple Music** — Apple does not expose this via the web API; we build our own signal from in-app behavior
- **Spotify integration** — not viable. As of Feb 2026, Spotify deprecated the endpoints a discovery app needs (audio features, audio analysis, related artists, recommendations, editorial playlists). New apps are also capped at 5 manually allowlisted users in "Development Mode," and "Extended Quota Mode" effectively requires an existing 250k+ MAU business (>95% of applications rejected per Spotify's own blog). This is a closed door, not a v2 consideration.
- **YouTube integration** — out of scope for v1
- **Native iOS or Mac app** — v2; v1 is a web app
- **Paid/managed AI tier** — v2; v1 is BYOK only
- **Social or collaborative features** — not in v1
- **Siri or voice input** — not in v1

---

## 6. Technical shape

- **Type of app:** Full-stack web app. Frontend served from Cloudflare Pages, backend logic in Cloudflare Workers.
- **Does it need to store data?** Yes — user taste profile, recommendation history, explicit feedback (liked/disliked/added), in-app listening session events.
- **Does it need authentication?** Yes. Users log in so their taste profile and history persist across sessions and devices. Auth is required (not optional) because the taste profile is the core value of the app.
- **Does it need to call external services?** Yes — MusicKit JS (Apple Music catalog + playback + library), and the user's own LLM API key (Google AI Studio / Gemini as the primary recommended option, others supported).
- **Who pays for hosting?** Developer (Carlos) pays for Cloudflare. Users pay for their own LLM API keys. Apple Developer account ($99/year) required for MusicKit.

### Proposed Cloudflare stack

| Need | CF Product | Why |
|---|---|---|
| Hosting the web UI | Pages | Free static + serverless hosting, deploys from git |
| Backend API | Workers | Serverless functions for taste profile, recommendations, session tracking |
| Structured data | D1 | SQL database for users, taste profiles, recommendation history, feedback events |
| Cross-device sync | R2 | Store library snapshot JSON for fast cold-start profile building |
| AI call proxying | AI Gateway | Proxies the user's own API key — adds logging and rate limiting for free |
| Session / auth tokens | KV | Fast global key-value store for auth tokens |

### MusicKit JS — known limitations

- Play counts are not accessible via the web API. Taste profile must be built from library contents + in-app session tracking + explicit feedback.
- Recently played returns albums/playlists only, not individual songs.
- Heavy rotation endpoint appears non-functional for third-party apps.
- Skip detection is inferred (track change before completion), not a native event. Expect ~1–2 second precision.
- Library fetch requires pagination at 100 songs per request. Handle 429 rate limit responses with backoff.
- You can only track listening behavior that happens inside your app.

---

## 7. Risks and unknowns

- **Biggest risk:** MusicKit JS is underdocumented and consistently behind the native Swift SDK. Expect to discover additional limitations mid-build. Allocate buffer time for API surprises.
- **Second biggest risk:** Understanding the code well enough to answer unscripted questions at demo. Build simply, use the rubber-duck-quiz skill before every commit, and favor code you can explain over code you can't.
- **Cold-start quality:** Library contents alone may not produce good enough recommendations to hook a new user before they've built up in-app feedback history. This needs to be tested early — week 2.
- **Things I don't know how to do yet:** MusicKit JS auth (developer tokens + music user tokens), Cloudflare Workers + D1 data modeling, LLM prompt engineering for music recommendations.
- **Things I'm assuming but haven't verified:** That library analysis (artist diversity, genre spread, era distribution) produces meaningful taste signals without play counts.

---

## 8. Milestones

- **Week 2 end:** MusicKit auth working, user library loaded and stored, basic chat UI deployed to Cloudflare Pages and publicly accessible. Cold-start recommendations generated from library contents and displayed as cards. This is the riskiest week — if MusicKit auth or library access hits a wall, surface it here, not in week 4.
- **Week 3 end:** LLM connected with user's own API key (Google AI Studio onboarding flow complete), recommendations responding to natural language mood/context input, like/dislike/add-to-library buttons functional and writing feedback to D1.
- **Week 4 demo:** Taste profile visibly improving recommendations based on accumulated feedback. App is publicly deployed, demo-ready, and the developer can answer unscripted questions about how every part works.
