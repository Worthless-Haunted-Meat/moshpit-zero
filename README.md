# MoshPit Zero

**Front-row energy, zero barriers.**

A program of [Worthless Haunted Meat](https://worthlesshauntedmeat.org), a 501(c)(3) nonprofit.

> Full MVP specification: [SPEC.md](SPEC.md)

## Goal

Give handicapped and mobility-limited concert-goers a **live** front-row experience of a show they are physically attending — starting with the metal cruise. A 180° camera on the stage rail (and one on the drum riser) streams to VR headsets in the accessible seating area, in real time, so the video matches the artist they're hearing live.

## Purpose

Wheelchair users at a packed metal show almost never get the rail — accessible viewing areas are usually far back or off to the side. The energy of being front row, or behind the drummer looking out at the crowd, is simply unavailable to them. MoshPit Zero exists to close that gap with cheap, off-the-shelf hardware and free software, and to publish the whole recipe (GPL-3.0) so any venue, festival, or fan can replicate it.

Design principles:

- **Live or nothing.** The video must stay in lip-sync with the artist the viewer hears in the room (< 1 s target, 3 s hard max). Delayed replay is a disaster fallback, not the product.
- **Turnkey.** No laptop, no browser fiddling, no operator console. Power on a sealed appliance; put on a headset; see the show.
- **Cost-effective.** Consumer cameras, Quest 3S headsets, a fanless mini-PC, and open-source software. Whole rig ≤ $3,000.
- **Self-contained.** Everything runs on an air-gapped local Wi-Fi network — no internet required, which is essential on a cruise ship.

## How It Works

```
[180° rail cam]   [180° drum-riser cam]
       └────────┬────────┘  (wired)
                ▼
   Fanless N100 appliance
   ffmpeg (HW encode) → MediaMTX (WebRTC)
                ▼
   Dedicated air-gapped Wi-Fi router
                ▼
   Quest 3S headsets in kiosk mode
   (wake → live rail view; one button toggles Rail ↔ Drum Cam)
```

## Estimated Cost (2-camera, 3-headset pilot)

| Piece | Est. cost |
|---|---|
| 2× 180° cameras (rail + drum riser) | $800–1,600 |
| 3× Meta Quest 3S | ~$900 |
| N100 appliance + capture dongles | ~$240–280 |
| Router, mounts, power, hygiene kits | ~$360 |
| **Total** | **~$2,400–3,000** |

Nonprofit hardware discounts (e.g. TechSoup) may reduce this.

## Development

**Current status: specification phase.** No hardware purchased, no code written yet.

Roadmap:

1. **Spec** — done, see [SPEC.md](SPEC.md) (v0.3: live-first, turnkey appliance, two cameras)
2. **Bench prototype** — prove the latency chain (camera → ffmpeg → MediaMTX WebRTC → Quest) hits < 1 s glass-to-glass before buying the full rig
3. **Appliance build** — headless auto-boot pipeline (systemd units), Quest kiosk configuration, in-player angle toggle
4. **Venue pilot** — one show, three viewers, measured lip-sync, logged results
5. **Publish** — setup scripts, parts list, and a step-by-step replication guide in this repo

Deferred until real demand signals appear (see SPEC §12): more headsets, side-stage cameras, spatial audio, a custom headset app.

## License

[GPL-3.0](LICENSE) — free to use, replicate, and modify; derivatives stay open.
