# MoshPit Zero

**Front-row energy, zero barriers.** Live VR front-row view for handicapped concert-goers on the metal cruise.

A program of [Worthless Haunted Meat](https://worthlesshauntedmeat.org), a 501(c)(3) nonprofit.

> Full MVP specification: [SPEC.md](SPEC.md)

## The Pitch

Wheelchair users and mobility-limited fans never get the rail. MoshPit Zero puts a 180° camera at the stage and streams a front-row POV to VR headsets anywhere in the venue — everyone gets a spot on the rail.

## MVP: one camera, three headsets, local Wi-Fi

| Piece | Gear | Est. cost |
|---|---|---|
| Capture | 1× 180° camera (Canon PowerShot V10-class 3D/VR cam or Insta360 X4), stage-side mount | $400–800 |
| Headsets | 3× Meta Quest 3S (passthrough lets users glance at surroundings without unstrapping) | ~$900 |
| Delivery | Dedicated travel router, local RTMP/WebRTC — no internet needed (ship bandwidth is scarce) | ~$100 |
| **Total** | 3-person pilot | **~$1,400–1,800** |

180° over 360°: half the bandwidth, and nobody looks behind the camera at a concert.

## Risk & fallback

Live low-latency 180° streaming is the hard part — needs a laptop encoding and a babysitter during shows.

**Zero-risk fallback:** record during the show, watch 20 minutes later. Same gear, near-zero failure modes, fine v1 signal. If people love the replay, that's the evidence the live rig is worth the complexity.

## Deferred (with go-signals)

- **More headsets** → when 3 users actually fight over them
- **Live streaming rig** → when replay mode gets genuine enthusiasm
- **Official pitch to cruise organizers** → when there's one cruise's worth of footage and reactions to show

## Open questions

- Which streaming software handles live 180° on a LAN reliably
- Camera mount / venue permission logistics on the ship
- Headset sanitizing + charging workflow between shows

## License

[GPL-3.0](LICENSE)
