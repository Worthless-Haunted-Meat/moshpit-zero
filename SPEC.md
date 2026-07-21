# MoshPit Zero — MVP Specification

Version 0.2 — pilot scope: one venue, one camera, three headsets, turnkey appliance.

## 1. Goal

Deliver a **live** front-row (rail) point-of-view of a concert to VR headsets worn by mobility-limited attendees elsewhere in the venue, over a self-contained local network with no internet dependency.

**Hard constraint — live sync:** viewers hear the venue's real sound while watching. Video must stay within lip-sync tolerance of the live artist (≤ 1 s target, 3 s absolute max). Any architecture that cannot hold this is rejected — this rules out camera-direct RTMP push (2–5 s) and HLS as a primary path.

**Turnkey constraint:** no laptop, no browser fiddling at show time. Operator powers on a sealed appliance; viewers put on a headset and see the show.

## 2. Non-Goals (v1)

- No internet streaming or remote viewers off-ship
- No multi-camera switching or production mixing
- No spatial/binaural audio capture (stereo shotgun or camera mic only)
- No more than 3 concurrent viewers
- No custom headset app — the Quest's built-in browser or an off-the-shelf VR video player only

## 3. Users

| Role | Description |
|---|---|
| Viewer | Wheelchair user or mobility-limited attendee, seated in an accessible area of the venue. May have never used VR. Must be able to start/stop viewing without assistance after a 2-minute orientation. |
| Operator | One person (initially the project owner). Sets up rig pre-show, monitors during, tears down after. |

## 4. System Overview

```
[180° camera @ stage rail]
        │ USB-C (UVC webcam mode) or HDMI → USB capture  ← wired, no camera Wi-Fi/RTMP
        ▼
[Appliance: fanless N100 mini-PC, headless]
  ffmpeg (QuickSync HW encode) → MediaMTX (WebRTC)
  auto-boots into the pipeline via systemd, ~60 s to live
        │ Ethernet
        ▼
[Dedicated travel router — isolated Wi-Fi, no internet]
        │
        ▼
[3× Quest 3S in kiosk mode — player auto-launches into the stream on wake]
```

Why not simpler options:

- **Camera RTMP push (no box at all):** 2–5 s latency — fails live sync. Rejected.
- **Raspberry Pi 5 as the appliance:** no hardware video encoder; can't do 4K encode. Rejected.
- **Laptop + OBS (v0.1 design):** works but not turnkey; kept only as the operator's bench-test rig.

## 5. Hardware

| Item | Spec | Qty | Est. |
|---|---|---|---|
| Camera | 180° 3D/VR capable, min 4K@30 per eye equivalent, USB webcam mode or clean HDMI out (Canon PowerShot V10-class VR cam or Insta360 X4 in 180° mode) | 1 | $400–800 |
| Headsets | Meta Quest 3S, 128 GB | 3 | ~$900 |
| Router | Dual-band travel router, WPA3, ≥ Wi-Fi 5; venue-isolated SSID | 1 | ~$100 |
| Appliance | Fanless Intel N100 mini-PC, 8–16 GB RAM, QuickSync HW encode; runs headless Linux | 1 | ~$150–200 |
| Capture | HDMI→USB 3.0 capture dongle (only if camera lacks a 4K UVC webcam mode) | 1 | ~$30 |
| Mount | Clamp or stand for rail/stage-side placement + safety tether | 1 | ~$50 |
| Power | Camera continuous power (USB-PD battery or venue outlet), headset chargers | — | ~$50 |
| Hygiene | Wipeable silicone facial interfaces + sanitizing wipes | 3 sets | ~$60 |

## 6. Software Pipeline

- **Capture:** camera in UVC webcam / HDMI-out mode at highest stable resolution ≥ 4K equirectangular 180°, wired into the appliance.
- **Encode:** ffmpeg with Intel QuickSync (h264_qsv), 4K@30, 20–30 Mbps CBR, zero-latency tune.
- **Distribute:** MediaMTX on the appliance, WebRTC only for live viewing.
- **View:** Quest 3S in kiosk mode — headset wakes directly into the 180° player (WebXR page in Quest browser kiosk, or DeoVR auto-launch) pointed at a fixed stream URL. Zero taps: don headset → see show.
- **Appliance lifecycle:** systemd units for capture/encode/serve with auto-restart; whole pipeline live within ~60 s of power-on with no interaction. Status LED or router-admin check is the only "monitoring UI" at show time.
- **Operator monitoring:** MediaMTX metrics endpoint viewable from the operator's phone on the same Wi-Fi (read-only status page — not part of the viewer path).

### Latency budget (hard requirement)

| Path | Target | Max acceptable |
|---|---|---|
| WebRTC end-to-end (glass-to-glass) | < 1 s | 3 s |

Viewers hear the venue live; video lag past ~3 s against the artist breaks the experience and is a pilot failure. There is no high-latency fallback for live viewing: if WebRTC can't hold 3 clients at 4K, reduce resolution (2880p → 1080p per eye) rather than switching protocols. HLS/RTMP are not acceptable live paths.

## 7. Network

- Dedicated router, own SSID (hidden), WPA3, no uplink — fully air-gapped from ship/venue networks.
- All devices on 5 GHz; router placed line-of-sight to the accessible seating area.
- Static IPs for laptop and camera; headsets via DHCP.
- Capacity check: 3 × 30 Mbps = 90 Mbps sustained — verify in venue with `iperf3` from each headset position before show one.

## 8. Modes

1. **Live mode (primary):** pipeline above, during the show.
2. **Replay mode (disaster fallback only):** camera records onboard; footage can be served from the appliance to headsets as on-demand 180° video after the fact. Replay is NOT the product — live sync with the artist is the product — but the recording always runs so total pipeline failure still yields something for viewers.

## 9. Operational Workflow

**Pre-show (T-60 min):** mount + tether camera, power camera + appliance + router, wait ~60 s for auto-boot, verify live stream on all 3 headsets from their actual seating positions (check lip-sync against a soundcheck or house music), start camera's onboard recording, sanitize facial interfaces, confirm headset batteries > 80%.

**During:** operator glances at the MediaMTX status page on their phone and does a headset spot-check every ~20 min. No laptop present.

**Post-show:** stop recording, offload footage, charge headsets, sanitize interfaces, log issues in `PILOT-LOG.md`.

## 10. Acceptance Criteria (pilot = success if all pass)

1. 3 headsets simultaneously stream the live 180° view for a full ≥ 60-minute set with ≤ 2 rebuffer events per headset.
2. Video-to-venue-audio lag ≤ 3 s (target < 1 s) as perceived from the accessible seating area — measured, not eyeballed: clap test on camera vs. headset view.
3. A first-time VR user sees the live show by simply putting the headset on — zero taps after orientation.
4. Appliance reaches "live and serving" within 2 minutes of power-on with no operator interaction.
5. Total spend ≤ $2,200.
6. At least one viewer says, unprompted, that they'd use it again.

## 11. Risks

| Risk | Mitigation |
|---|---|
| Wi-Fi congestion in a metal-show crowd (thousands of phones) | 5 GHz, DFS channels, router near viewers; pre-show iperf3 check; HLS fallback |
| Camera overheating during 60+ min set | Continuous power + airflow; onboard recording as backup even if webcam feed dies |
| Venue/artist filming restrictions | Get organizer permission BEFORE the cruise; camera is stationary and non-commercial |
| Motion sickness in viewers | Stationary camera only (no pans/zooms); 180° fixed POV is the low-nausea case; brief users on removing headset if uncomfortable |
| Live pipeline fails mid-show | Replay mode is always recording; worst case viewers watch the set 20 min delayed |

## 12. Deferred (with go-signals)

- **More headsets** → 3 users actually competing for them
- **Spatial audio** → viewers report venue audio is insufficient
- **Multi-camera / switching** → viewers ask for different angles
- **Custom headset app** → kiosk-mode player proves too fragile across Quest OS updates
- **Official cruise-organizer program** → one cruise of footage + viewer reactions in hand
