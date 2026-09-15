# MedTalk Call Desk

Live status board for **remote cross-platform MedTalk call tests** — an Android phone on one
desk, an iPhone on another, calling each other while both halves report here.

**Open it:** https://locoDevKumR.github.io/medtalk-call-desk/

It is a single static page. It stores nothing and ships no data of its own: it polls a
**relay** that runs on the Android tester's Mac and renders what the two runners post to it.

## Using it

1. The Android side starts the relay and exposes it:
   ```
   MEDTALK_RELAY_TOKEN=<secret> python3 scripts/medtalk-relay-server.py
   cloudflared tunnel --url http://localhost:8787
   ```
2. Open this page and paste the tunnel URL into **Relay**, or share a pre-filled link:
   `https://locoDevKumR.github.io/medtalk-call-desk/?relay=https://xxx.trycloudflare.com`
3. Both testers run their half with the **same `RUN_ID`**. The board fills in as they go.

The relay also serves this same page at its own root, so `https://xxx.trycloudflare.com/`
works with no setup at all and pre-fills its own URL.

## What it shows

- **Four media paths** — video and audio, in both directions. Each needs to *advance*
  between samples, not merely be non-zero: a frozen keyframe and a stalled decoder both
  satisfy `> 0`.
- **Per-device phases and logs** — so a failure on the far device is readable from either desk.
- **Liveness** — each side heartbeats every 15s. A side that stops reporting is called out
  ("no heartbeat for 71s", "LOST CONTACT") instead of sitting on "RUNNING" forever.
- **Relay health** — an unreachable relay is named as such, with the last known data kept
  on screen rather than blanked.

Reading a red run: **a rendezvous timeout names the healthy device** — the side that timed
out was working, so read the other column first. Exit code 2 is infrastructure (device or
relay gone), never a product defect.

## Privacy

Public page, no analytics, no external calls except Google Fonts. The relay URL is typed in
by the viewer and kept in that browser's `localStorage`. Test credentials never pass through
here — they stay in the runners' own environment.
