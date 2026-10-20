# OpenStick.tv PRD

Oct 20, 2026 · @Chris · Revision 2

> Revision 2 changes: price removed as a goal and constraint; parity matrix recounted with an explicit verdict column; v1 board spike widened to full RK3588 and Intel N100-class x86 after finding the CM5 has no VP9 or AV1 decode; Chrome (not Chromium) made the single browser path and its packaging reconciled with A/B updates; YouTube treated as its own risk; new sections for users and positioning, success metrics, first-run setup, recovery, display and audio, guest mode, compliance, and continuity. Claims added in this revision that still need a source are marked **(verify)**. Revision 1 is kept as `PRD.original.md`.

## Summary

OpenStick.tv is an open-hardware, open-software TV streaming device that matches a Google TV Streamer on casting, CEC, 4K output, smart-home control and longevity, and is honest about the one thing it cannot do: play Netflix, Disney+, HBO Max, Prime or Apple TV+ above SD to 720p. That ceiling is set by Widevine L1 and per-service certification, which no open device can obtain, and it is the same ceiling every Linux box, Raspberry Pi and uncertified Android stick already lives with.

Four findings from the research shape the plan:

1. The Orange Pi Zero 2W is a v0 prototyping board only. Its HDMI output is not in mainline Linux, its hardware decoder is not usable on the vendor kernel, and it has no AV1. v1 is chosen by a hardware spike across a Pi CM5 stick carrier, a full RK3588 board, and an Intel N100-class x86 box.
2. 4K YouTube needs VP9 or AV1 hardware decode. The CM5 has neither, so it cannot be the 4K reference despite being the only true stick. That forces a product decision: **stick form factor, or 4K YouTube everywhere.** See Hardware.
3. The DRM story is software Widevine L3 through official Google Chrome for arm64 or x86-64 Linux. Expect 720p on Netflix, Disney+ and HBO Max, SD on Prime, unknown on Apple TV+. The Kodi Netflix add-on is dead. The Android TV route gives the same SD ceiling with a worse stack.
4. Google Cast is not reachable: stock phones reject uncertified receivers and no licence is offered. The open substitutes are FCast, AirPlay (UxPlay), Miracast, DIAL and DLNA, and Netflix itself dropped phone casting for most devices in December 2025, which shrinks the gap. YouTube, the most important remaining service, has its own fragility (see Streaming).

The differentiators that remain are real: guest casting Google removed, Apple Home and Matter control of the TV, Home Assistant integration, no telemetry, a local API, five years of signed updates, and a website that teaches the hardware through interactive 3D exploded views.

## Users and positioning

**Primary users**

- **Self-hoster.** Runs Jellyfin, Plex or Home Assistant. Wants a TV endpoint that is controllable, private and not ad-driven. Tolerates 720p Netflix.
- **Privacy-minded household.** Wants a streaming device that does not profile them, with guest casting and an Apple-or-Google-agnostic remote.
- **Hobbyist builder.** Wants to understand and build the device; the website and build guide are the product for this user.

**Not the user:** someone whose main viewing is 4K Netflix or Disney+. The site says so on the home page.

**Alternatives this is measured against**

| Alternative | Why someone picks it | What OpenStick must do better |
| --- | --- | --- |
| Google TV Streamer, Fire TV, Roku | Certified DRM, 4K everything | Privacy, local control, guest casting, no ads, longevity |
| LibreELEC or OSMC on a Pi 5 | Open, mature Kodi, large community | Casting receivers, smart-home integration, Chrome web-app layer, signed A/B OTA, stick form, finished UX |
| OSMC Vero V | Polished open Kodi box with support | Casting and smart-home; open hardware files |
| Nvidia Shield | Best Android TV box, certified | Open stack, long updates without a vendor |

If OpenStick is not clearly better than "LibreELEC on a Pi 5" for the self-hoster, it has no reason to exist. The casting receivers, Home Assistant and Matter integration, and the 10-foot launcher are what close that gap.

## Goals and non-goals

OpenStick ships an open-hardware, open-software streaming device that does what a Chromecast does for the things an open device can legally do, and says plainly what it cannot.

**Goals (v1)**

- Plug-and-play HDMI device, wall-powered (TV USB power is not reliable for 4K decode or CEC wake), 4K60 output, HDMI-CEC so the TV remote works on day one.
- First-run setup with no keyboard: Wi-Fi joined from a phone or the TV remote in under five minutes.
- Casting receiver for phones and laptops: AirPlay, Miracast, FCast, and DIAL-style app launch, with Google Cast handled through the best open path available.
- A 10-foot UI with a launcher, app model, settings, and OTA updates, built on a mainline-Linux image that anyone can rebuild from source.
- Streaming apps for the major services at the best quality open hardware is permitted to deliver, with the resolution ceiling shown honestly in the app tile.
- Recovery from any failed update or bad config without a computer.
- Fully published hardware: BOM, enclosure CAD, carrier/expansion board files, and a build guide a hobbyist can follow in an afternoon.
- A website that teaches the product through interactive 3D exploded views and animated streaming diagrams, and doubles as the build guide.

**Non-goals (v1)**

- Matching Chromecast on DRM-gated 4K from Netflix, Disney+, HBO Max or Apple TV+. That needs Widevine L1 plus per-service certification, which is not available to an open device. Streaming services and DRM covers what is reachable.
- Google Assistant, Google TV recommendations, or any feature that depends on Google Play Services.
- Custom silicon or a custom SoC carrier board. v0 and v1 use an off-the-shelf board; a custom carrier is a v2 question.
- Being a general-purpose desktop. The image boots to the 10-foot UI; a shell is available but not the product.
- A mobile companion app beyond what casting protocols and the local web UI already provide.
- Price as a design constraint. Parts are chosen for capability; the BOM is published for transparency, not as a target.

## Success metrics

| Metric | Target for v1 GA | How it is measured without telemetry |
| --- | --- | --- |
| First-run success | 90% of beta testers reach the home screen without help | Beta survey and observed setup sessions |
| Update reliability | Zero devices bricked by OTA; rollback works in 100% of injected-failure tests | CI fault-injection on real hardware; opt-in crash reports |
| Service availability | Every "supported" tile passes the weekly playback check | Automated playback lab (see Streaming) |
| Casting coverage | YouTube via DIAL, AirPlay, FCast and DLNA work from a stock phone in one tap | Release checklist on a device matrix |
| Community | Build guide completed by 50 beta builders; median build time under 3 hours | Beta survey |
| Longevity | Security updates shipped within 14 days of a critical upstream CVE | Public advisory log |

## Chromecast parity matrix

Of the 19 capabilities compared against a Google TV Streamer, OpenStick v1 matches 8, does better on 2, partially matches 6, and cannot do 3 (Google Cast, Assistant, DRM-gated 4K). The benchmark is the [Google TV Streamer (4K, 2024)](https://store.google.com/product/google_tv_streamer_specs), with the 2020 Chromecast with Google TV as the floor.

| Capability | Google TV Streamer | OpenStick v1 | Verdict | How |
| --- | --- | --- | --- | --- |
| 4K60 output | Yes | Yes | Match | All v1 candidates output 4K60 on mainline; H618 only on vendor kernel |
| HDR10 / HDR10+ / Dolby Vision / HLG | All four | HDR10 and HLG, no DV | Partial | Dolby Vision is licensed; HDR10 metadata passthrough unproven on H618 under Linux, expected on RK3588, Pi 5 and Intel **(verify in spike)** |
| Dolby Atmos / DD+ | Yes | Passthrough only, native apps only | Partial | Kodi passes bitstreams; Chrome web apps output decoded stereo or PCM multichannel, never passthrough |
| Google Cast receiver (video, audio) | Yes | No | No | Cast receiver requires a Google license; see Software for the open substitutes |
| Screen and tab mirroring | From Android and Chrome | Yes | Match | Miracast (Android, Windows) via MiracleCast; AirPlay mirroring via UxPlay; Chrome tab via WebRTC page |
| Launcher / home with recommendations | Google TV | Launcher, no recommendations | Partial | Own 10-foot UI; no Google content graph |
| Voice remote and Assistant | Yes, plus Gemini | No (v1), partial (v2) | No | BLE remote mics use vendor GATT audio with no Linux receiver; v2 option is phone-as-mic or USB mic with an open STT model |
| HDMI-CEC: TV power on, input switch on cast | Yes, wall power required | Yes | Match | libCEC; working on CM5 and expected on RK3588 and Intel boards with CEC wired; unproven on H618 |
| Ethernet | Built-in 1 Gbps | 1G on v1 boards | Match | 100M on the v0 expansion board |
| Bluetooth remote, game controllers, headphones | BT 5.1 | Yes | Match | BlueZ; BT 5.0 to 5.4 depending on board |
| Wi-Fi | 802.11ac dual band | Wi-Fi 5 or 6 by board | Match |  |
| Ambient mode / screensaver with photos | Google Photos | Yes, any source | Match | Local, S3, Immich, or Nextcloud photo sources |
| Multi-user and kids profiles | Yes | Simpler | Partial | Profile switcher with PIN; can hide apps per profile but cannot enforce ratings inside third-party web apps |
| Find my remote | Button and voice | With a buzzer-equipped remote | Partial | Most BLE remotes lack a buzzer; v1 ships one that has it or drops the feature |
| Guest mode | Removed by Google in 2023 | Yes | Better | On-screen PIN or QR pairing; see Guest mode |
| Matter and Thread border router | Yes | Matter device v1, Thread v2 | Partial | Exposes a Matter video player; Thread needs a radio no candidate board has; Matter certification is a cost and process (see Ecosystem) |
| Five years of OS updates | Yes | Yes by design | Match | Signed OTA with A/B rollback; continuity plan in Risks |
| Streaming apps at 4K with DRM | Yes, certified | No at 4K | No | See Streaming services and DRM |
| USB for storage or wired accessories | None, needs a hub | Yes | Better | USB-A on the board, carrier or expansion board |

What Google gets from its chip that some candidate boards do not: AV1 and VP9 decode (YouTube 4K), a certified DRM TEE, and Dolby Vision. Only the first is solvable by board choice, and it drives the v1 decision.

Sources: [Google spec table](https://support.google.com/chromecast/answer/3046409), [Google CEC page](https://support.google.com/chromecast/answer/7199917), [Streamer launch post](https://blog.google/products/google-nest/google-tv-streamer/), [PCWorld review](https://www.pcworld.com/article/2494628/google-tv-streamer-review.html), [TechRadar on guest mode removal](https://www.techradar.com/televisions/streaming-devices/google-just-removed-one-of-chromecasts-most-useful-features).

## Streaming services and DRM

OpenStick gets software Widevine L3 and nothing higher, so the major subscription services top out at SD to 720p and 4K is off the table on any open board. The product promise is YouTube, self-hosted media, open casting, and "your subscriptions in a browser at reduced quality."

**Why: the three locks**

1. Widevine L1 needs a secure bootloader, a factory-provisioned device-unique keybox, and decryption inside a TEE, plus a signed agreement and robustness review with Google ([Widevine](https://www.widevine.com/solutions/widevine-drm), [Khadas thread](https://forum.khadas.com/t/widevine-l1-for-khadas-vim/885)). No open-hardware project has obtained it. Custom ROMs that keep L1 (GrapheneOS on Pixel) ride the phone vendor's existing keybox, which does not transfer to a hobby board.
2. Each service then certifies devices separately. Netflix sends uncertified-device users to a Play Protect certified device ([Netflix](https://help.netflix.com/en/node/124156)); uncertified Android boxes get SD at best and are sometimes blocked outright.
3. Google Cast senders verify a Google-issued certificate on the receiver, so a third-party Cast receiver is rejected by stock phones ([gcast-auth](https://yingtongli.me/blog/2019/12/20/gcast-auth.html)). No public receiver licence exists. Netflix also removed phone casting for most devices in December 2025 ([Android Authority](https://www.androidauthority.com/netflix-casting-chromecast-google-tv-streamer-3620784)), so Cast parity matters less than it did.

**The browser path: Google Chrome, not Chromium**

The Widevine CDM is only legally available inside Google Chrome, so the web-app layer is Google Chrome in kiosk mode, not Chromium with a borrowed CDM. Google ships official Chrome for arm64 Linux with a native Widevine L3 module, formally supported since August 2026 ([OMG Ubuntu](https://www.omgubuntu.co.uk/2026/07/chrome-arm64-linux-available)), and has long shipped x86-64 Linux Chrome. Kodi's InputStream Adaptive can load the same CDM, but the CastagnaIT Netflix add-on is suspended and effectively broken since Netflix changed its licence flow ([issue #1627](https://github.com/CastagnaIT/plugin.video.netflix/issues/1627)).

Consequences:

- **Packaging.** Chrome ships as a Debian package with glibc and Debian library dependencies, and its licence does not allow OpenStick to redistribute it. The image therefore uses a Debian userland (see Software) and installs Chrome from Google's apt repository on first run, with the user accepting Google's terms on screen.
- **Hardware decode inside Chrome.** Chrome on desktop Linux accelerates video through VA-API (Intel, AMD). The V4L2 decoders on Rockchip and Broadcom SoCs are, as far as current research shows, only used by ChromeOS builds **(verify in spike)**. On ARM, video inside Chrome is likely software-decoded. That is fine for L3 720p but rules out 4K YouTube in the browser on ARM.
- **Support horizon.** Chrome for arm64 Linux is new. If Google drops it, the web-app layer on ARM goes with it; x86-64 Linux Chrome has no such risk.

**YouTube**

YouTube is the most-used service on the device and the path is less settled than the table used to imply.

| Path | Status | Risk |
| --- | --- | --- |
| youtube.com in Chrome (desktop site) | Works; 10-foot UX is poor | Low; quality capped by in-browser decode on ARM |
| youtube.com/tv (TV interface) in Chrome | Provides the 10-foot UI and DIAL/"Link with TV code" pairing | YouTube restricts this interface to approved TV clients; using it from a browser needs a TV user-agent **(verify current behaviour and terms)** |
| Kodi YouTube add-on | Hardware decode, 4K where the board decodes VP9/AV1 | Each user must create their own API key; breaks when YouTube changes client attestation |
| DIAL launch from the phone app | Depends on one of the above as the receiving app | Inherits the receiver's risk |

Decision needed in the v0 phase: which YouTube path is the default, and whether OpenStick seeks YouTube's device partner programme (unlikely to be granted, but the answer should be on record).

**Service table**

| Service | Path on OpenStick | What you get | Confidence |
| --- | --- | --- | --- |
| YouTube | See YouTube above | Up to 4K in native playback on boards with VP9/AV1 hardware decode; 1080p or lower in Chrome on ARM | Medium |
| Jellyfin, Plex, Emby, Kodi library | Native | Full quality, hardware decode | High |
| Netflix | Chrome web app, L3 | 720p likely, 1080p claimed by [Netflix's help page](https://help.netflix.com/en/node/30081), which also says Linux is unsupported | Medium; no authoritative 2026 ARM test |
| Disney+ | Chrome, L3 | 720p, no 4K on any computer browser | Medium; intermittent 403 reports |
| HBO Max | Chrome | 720p is the [cap for the website](https://help.hbomax.com/la-en/Answer/Detail/000002521) on any device | Medium |
| Prime Video | Chrome, L3 | SD (480p to 540p); Amazon requires L1 for anything higher | High |
| Apple TV+ | tv.apple.com in Chrome | Undocumented; no non-browser path exists; AirPlay from the Apple TV app passes audio only, UxPlay cannot decrypt Apple video DRM | Low |
| Hulu, Paramount+ | Chrome | Unverified | Low |
| Peacock | Chrome | Actively blocking Linux browsers as of March 2026 | Blocked |

**The Android TV option, and why not**

Orange Pi ships an Android 12 TV image, but it is not Play Protect certified and carries no L1 keybox. Sideloaded Netflix on an uncertified box gets SD or nothing. No LineageOS or Android TV community build exists for the H618. Android would trade an open stack for an unmaintained vendor image and the same SD ceiling. Not worth it.

**Product rules**

- Every service tile shows its real ceiling ("Netflix, up to 720p") and the setup page says why, in two sentences, with a link to this reasoning.
- No service is listed as "supported" until the weekly automated playback check passes for 30 days; Peacock-style blocks happen without notice.
- The web-app layer is official Google Chrome with its bundled CDM, auto-updated from Google's repository, never an extracted ChromeOS or Chrome blob.
- If a certified-DRM tier is ever wanted, it is a separate product on a vendor-certified Android TV SoC and is out of scope for OpenStick's open image.

**Automated playback lab**

The weekly check is a real system and needs its own design:

- One device per supported board on a test bench, each with an HDMI capture card so the delivered resolution is measured from the actual output, not reported by the player.
- Paid test accounts for each service. Automating playback may break a service's terms of use; keep it to low-frequency checks (one title, two minutes, weekly) and review each service's terms before adding it **(verify per service)**.
- Results published on the website as the source for each tile's status.

**Open items**

- [ ] Run Netflix, Disney+, HBO Max, Prime and Apple TV+ in official Chrome on each spike board and record the resolution each delivers, measured by HDMI capture.
- [ ] Confirm whether Chrome on Linux arm64 uses any hardware video decode on RK3588 and CM5.
- [ ] Confirm the arm64 CDM runtime requirements (glibc TLS patch, LSE atomics) against the chosen image ([ISA wiki](https://github.com/xbmc/inputstream.adaptive/wiki/Widevine-ARM64-support)).
- [ ] Pick the default YouTube path and document the terms-of-service position.

## Hardware

The Orange Pi Zero 2W is a v0 prototyping board, not the v1 reference: its HDMI output is not in mainline Linux, hardware video decode on Linux is unfinished, and the Allwinner H618 has no AV1 decoder. v1 is chosen by a hardware spike.

**The form-factor decision: stick, or 4K everywhere**

No candidate gives both.

- **Stick:** the Pi CM5 on the Makerfabs TV Stick Lite is the only true stick with an HDMI plug and mainline Linux. Its BCM2712 hardware-decodes HEVC only: no VP9, AV1 or H.264. 4K YouTube is not achievable.
- **4K everywhere:** a full RK3588 board decodes HEVC, VP9 and AV1 in hardware on mainline, but Chrome likely cannot use that decoder. An Intel N100-class box decodes all three and Chrome uses it through VA-API, so 4K works in native apps and in the browser. Neither is a stick; both are small boxes with an HDMI cable.

Recommendation: decide after the spike, with a lean toward **an Intel N100-class reference** if browser-based YouTube and the Chrome path turn out to matter most, and toward **RK3588** if Kodi-based playback is the primary path. Keep the CM5 stick as a supported "compact" variant without the 4K YouTube claim.

**Board candidates**

| Board | SoC | RAM | HDMI out | Decode on Linux | Wi-Fi / BT | Verdict |
| --- | --- | --- | --- | --- | --- | --- |
| [Orange Pi Zero 2W](http://www.orangepi.org/html/hardWare/computerAndMicrocontrollers/details/Orange-Pi-Zero-2W.html) | Allwinner H618, 4x A53 1.5 GHz | 1 to 4 GB | 4K60, mini HDMI | HEVC 4K30 on vendor Android only; mainline Cedrus for H616 posted May 2026, not merged; no AV1 | Wi-Fi 5 / BT 5.0 | v0 prototype only |
| [Banana Pi BPI-M4 Zero](https://ameridroid.com/products/banana-pi-m4-zero) | H618 | 2 to 4 GB | 4K60, mini HDMI | Same as above, adds 8 GB eMMC | Wi-Fi 5 / BT 4.2 | No advantage |
| [Radxa Zero 3W](https://ameridroid.com/products/radxa-zero-3w) | RK3566 | 1 to 8 GB | 1080p60 on the spec sheet, micro HDMI | Good Rockchip VPU support | Wi-Fi 6 / BT 5.4 | Fails the 4K requirement |
| [Radxa ROCK 5C Lite](https://www.cnx-software.com/?p=130106) | RK3582 (cut-down RK3588S) | 1 to 16 GB | 4K-class HDMI, full-size port | RK3582 may ship with the GPU and parts of the VPU disabled **(verify)** | Wi-Fi 6 / BT 5.4 | Only if the RK3582 keeps GPU and VPU; otherwise use a full RK3588S |
| Full RK3588 / RK3588S board (e.g. ROCK 5B, Orange Pi 5 class) | RK3588 / RK3588S | 4 to 32 GB | 4K60 (8K on some), full-size port | HEVC, VP9, AV1 hardware decode, maturing in mainline; Mali GPU via Panthor | Board-dependent, often M.2 module | v1 candidate (4K, native apps) |
| Intel N100 / N150 mini PC or SBC | Alder Lake-N | 8 to 16 GB | 4K60, HDR | HEVC, VP9, AV1 via VA-API, used by Chrome | Wi-Fi 6 / BT 5.x typical | v1 candidate (4K, native and browser) |
| Pi CM5 + [Makerfabs TV Stick Lite](https://liliputing.com/turn-a-raspberry-pi-cm5-into-a-tv-stick-with-this-carrier-board/) | BCM2712 | 2 to 16 GB | HDMI plug built into the carrier | HEVC 4K60 only; no VP9, AV1 or H.264 hardware decode | Wi-Fi 5 / BT 5.0 | v1 compact variant, true stick, no 4K YouTube |
| [Raspberry Pi Zero 2 W](https://www.raspberrypi.com/products/raspberry-pi-zero-2-w/) | BCM2710 | 512 MB | 1080p | 1080p30 | Wi-Fi 4 / BT 4.2 | Fails 4K |

**Spike exit criteria (every board)**

1. 4K60 HEVC and 4K VP9 and AV1 playback in Kodi with hardware decode, measured dropped frames.
2. Same content in Chrome; record whether decode is hardware or software.
3. HDR10 metadata reaches the TV (TV info overlay shows HDR).
4. CEC: power on, input switch, volume pass-through with `cec-ctl`.
5. Sustained 30-minute 4K playback inside the closed enclosure; record SoC temperature and throttling.
6. Power draw at idle and peak; confirm the wall adapter rating.
7. Netflix, Disney+, HBO Max resolution in Chrome via HDMI capture.
8. Availability: stock at two or more distributors and stated lead time.

**v0 BOM (Orange Pi Zero 2W)**

| Part | Spec | Note |
| --- | --- | --- |
| Orange Pi Zero 2W, 2 GB | H618, LPDDR4, Wi-Fi 5, BT 5.0, 16 MB SPI flash, microSD | 2 GB minimum for Chrome |
| [24-pin expansion board](https://www.cnx-software.com/2023/09/09/orange-pi-zero-2w-raspberry-pi-zero-2w-alternative-4gb-ram/) | 2x USB-A, 100M Ethernet, IR receiver, 3.5 mm audio, power and user buttons | Required for IR and USB-A |
| microSD, 32 GB A1 | Boot and app storage | No eMMC on this board |
| Mini HDMI male to HDMI male adapter | Short rigid adapter forming the plug | Verify gender before buying; most sold adapters are the wrong way round |
| Wall adapter, 5 V 2 A USB-C | Board is rated 5 V 2 A, about 10 W peak | A TV USB port (2.5 to 4.5 W) is not reliable; Google requires wall power for CEC on Chromecast too |
| Heatsink, 20 mm with thermal pad | Passive | Sibling Zero 3W hit 63 C at 60 s all-core; no sustained 4K data exists, so include it |
| Enclosure | 3D-printed two-part shell | No existing stick case for this board; custom design required |
| BLE or 2.4 GHz remote | See Remote |  |

The v1 BOM is written after the spike. CM5 note: the CM5 can draw well above 10 W under load **(verify on the carrier)**, so the stick variant needs a 5 V 3 A or higher wall adapter and a thermal design validated in step 5.

**HDMI-CEC**

Unproven on H618. The SoC has a DesignWare HDMI TX with a hardware CEC block, and `dw_hdmi_cec` loaded on a patched Armbian 6.6 kernel, but no one has reported `cec-client` or libCEC working on H616/H618 ([linux-sunxi status](https://linux-sunxi.org/Linux_mainlining_effort) shows "?"). Test on v0 hardware for information, but it does not block the project: the CM5 has CEC working today, and RK3588 and Intel boards are checked in the spike. On Intel boards CEC is only present if the board vendor wired it or a Pulse-Eight USB-CEC adapter is added **(verify per board)**.

**Remote**

1. TV remote via CEC: zero extra hardware if CEC works.
2. [MX3-class 2.4 GHz air mouse](https://www.cnx-software.com/?p=42407): USB HID, works on any Linux, needs a USB-A port.
3. G10S Pro-class BLE remote: BLE HID for keys works through BlueZ. Voice does not: remote microphones use vendor-specific GATT audio, and no Linux receiver exists. Voice in v1 means phone-as-remote or a USB mic, not the remote.
4. IR via rc-core/LIRC: works for any universal remote; needs an IR receiver on the carrier, expansion board or a USB IR dongle on boards without one.

Decision for v1: one shipped remote, chosen after the spike, that works with every v1 board without an adapter.

**Decode and HDR facts for the H618** ([CNX spec sheet](https://www.cnx-software.com/2022/07/30/allwinner-h618-processor-powers-android-12-tv-boxes/)): HEVC Main10 and VP9 to 6Kp30, H.264 to 4Kp30, no AV1, HDMI 2.0 with HDR10 and HDCP 2.2 in silicon. On Linux: 1080p50 YouTube at 20% CPU with ffmpeg v4l2-request on Manjaro; HDR metadata has never been shown passing to the TV on Linux; the community LibreELEC port had HEVC artifacts and no HDMI audio ([LibreELEC thread](https://forum.libreelec.tv/thread/27650-orange-pi-zero-2w/)). Vendor Android 13 plays 4K30 with hardware decode.

**Supply**

Several candidates were out of stock during research. The v1 reference must be in stock at two or more distributors, and the build guide names a second-source board for each part where one exists.

**Open items**

- [ ] Order a CM5 + Makerfabs TV Stick Lite, a full RK3588S board, and an N100-class box for the spike.
- [ ] Confirm whether the RK3582 in the ROCK 5C Lite has GPU and VPU enabled.
- [ ] Test CEC on the Zero 2W with `cec-ctl` on the Armbian patched kernel (informational).
- [ ] Run the eight spike exit criteria on each board.
- [ ] Make the stick-vs-4K decision and record it in `decisions/`.

## Software platform

OpenStick OS is a signed, A/B-updatable Linux image on a Debian userland with a mainline (or near-mainline) kernel, booting straight into a 10-foot launcher that hosts three app types: native (Kodi, Jellyfin), web (Google Chrome in kiosk mode), and casting receivers.

**Layers**

| Layer | Choice | Note |
| --- | --- | --- |
| Build | Debian-based image built with debos or mkosi **(decide)** | Buildroot was the original pick, but Google Chrome is a Debian package with Debian library dependencies; a Debian userland avoids maintaining a compatibility layer. Reproducible from one repo; LibreELEC's build discipline is still the model |
| Kernel | Mainline plus the minimum patch set the board needs | On H618, out-of-tree HDMI and Cedrus patches; on CM5, RK3588 and Intel close to clean mainline |
| Display | DRM/KMS, Wayland (wlroots-based compositor such as cage) | No X11; the compositor owns the HDMI output and hands it to whichever app is in front; UI renders at 1080p and video planes scale to 4K |
| Media | GStreamer and ffmpeg with v4l2-request (ARM) or VA-API (Intel); Kodi for the library |  |
| Audio | PipeWire with WirePlumber | HDMI passthrough profiles for Kodi; Chrome gets PCM |
| Launcher | Own 10-foot UI, Flutter or web **(decide)** | Web keeps one UI stack for launcher, settings and web apps; Flutter is leaner on RAM |
| Web apps | Google Chrome kiosk, one isolated profile per service | Service tiles declare their resolution ceiling |
| Casting | FCast receiver, UxPlay for AirPlay, MiracleCast for Miracast, DIAL server, DLNA renderer | FCast includes a Google Cast receiver disabled by default; stock senders reject it, so it stays off |
| Device daemon | `openstickd`: settings, app manifests, local API, CEC, HAP and Matter bridges | One process owns device state; UIs and integrations talk to it |
| Updates | Signed OTA, A/B partitions, automatic rollback, RAUC **(or Mender; decide)** | Five-year support promise depends on this |
| Remote input | libCEC, BlueZ HID, rc-core for IR, local web remote | Phone-as-remote via the local web UI needs no app |
| Local API | REST plus WebSocket over mDNS, token auth | Feeds Home Assistant and the Matter bridge |

**Partition layout and Chrome**

- `boot`, `rootfs-A`, `rootfs-B`: read-only, signed, swapped by RAUC.
- `vendor`: writable, holds Google Chrome installed from Google's apt repository on first run. Updated by a separate `chrome-updater` service that keeps the previous version for rollback. This keeps Chrome out of the redistributed image and lets it update on Google's schedule.
- `data`: user settings, app data, Flatpak apps. Survives OS updates; wiped by factory reset.

**First-run setup**

1. Boot shows a language picker driven by the CEC remote or any connected remote.
2. Wi-Fi: the screen shows a QR code. Scanning it joins the phone to a temporary `OpenStick-XXXX` access point and opens a setup page to choose the home network. Ethernet skips this step. A typed fallback with an on-screen keyboard is always available.
3. Accept Google's terms and install Chrome (skippable; tiles that need Chrome show "not installed").
4. Optional: name the device, pair a remote, enable Home Assistant / HomeKit / Matter.
5. Target: home screen in under five minutes.

**Recovery and reset**

- Automatic: a boot counter falls back to the other slot after three failed boots.
- Manual recovery: holding the board's button (or a key on the remote during boot) boots a minimal recovery slot offering "roll back", "factory reset", and "re-download OS".
- Factory reset wipes `data` and `vendor`; the signed OS slots are untouched.
- Last resort: the build guide documents reflashing from a computer.

**Display and audio handling**

- Match refresh rate to content (23.976/24/25/50/60) in Kodi and native players; Chrome stays at the desktop rate.
- Switch HDR mode only while HDR content plays; return to SDR for the UI.
- Read EDID to offer only supported modes; overscan adjustment in settings for older TVs.
- Audio passthrough settings per output (AC-3, E-AC-3, TrueHD, DTS) detected from EDID; lip-sync offset adjustable per output.
- CEC volume control routes to the TV or AVR via ARC where supported.

**App model**

Apps are Flatpak bundles with a manifest: name, icon, launch command, input mode (pointer, d-pad, keyboard), resolution ceiling, and required permissions (network, USB, mic). The launcher reads the manifest; the store is a static JSON index on openstick.tv plus any user-added repo URL.

**Guest mode**

Google's version used a PIN broadcast over ultrasound and a Wi-Fi beacon. OpenStick's version:

- By default, casting receivers accept senders on the same network only after the device is "paired" once.
- Guest mode, when enabled, shows a QR code and a 4-digit PIN on screen. A guest scans the QR code to join a temporary access point hosted by the stick (isolated from the home LAN) and cast through it, or enters the PIN in a sender that supports it.
- Guest sessions end when the TV turns off or after a set time. The owner can disable guest mode entirely.

**Accessibility and languages (on the TV)**

- Screen reader for the launcher and settings (Orca or a built-in TTS reader).
- High-contrast theme and adjustable text size.
- Captions on by default in native players when the profile asks for them.
- UI strings externalised from day one; English at launch, community translations after.

**Security baseline**

- Read-only root, verified boot where the board supports it (Secure Boot on Intel, signed U-Boot FIT on ARM), apps sandboxed (Flatpak portals), SSH off by default with a toggle in settings, no telemetry.
- Local API and web UI: device pairing by on-screen code issues the token; every request checks the `Host` header against the device's own names and addresses (blocks DNS rebinding); state-changing endpoints require the token, never cookies alone (blocks cross-site requests).
- Update signing key in a hardware token, with an escrowed backup key (see Continuity).
- Vulnerability disclosure via `SECURITY.md` and GitHub private advisories; critical upstream CVEs patched within 14 days.
- Opt-in crash and update-failure reports, off by default, sent to a self-hosted endpoint, contents shown before sending.

**Open items**

- [ ] Pick Flutter vs web for the launcher after a RAM and cold-start test on the smallest v1 board.
- [ ] Pick debos vs mkosi; confirm Google Chrome installs and updates cleanly into a separate `vendor` partition.
- [ ] Confirm Wi-Fi Direct support on the v0 and v1 radios for Miracast.
- [ ] Decide RAUC vs Mender for OTA.
- [ ] Prototype the QR-code Wi-Fi onboarding flow.

## Ecosystem integrations

OpenStick appears as a TV accessory in Apple Home, Google Home, Alexa, SmartThings and Home Assistant through two protocols (HomeKit Accessory Protocol and Matter) and one local API, so a user can say "turn on the living room TV" in whatever assistant they already own. It cannot be an Apple Home hub.

**Apple Home**

- Hub role: no. Only Apple TV and HomePod can act as the home hub that runs automations and gives remote access. There is no third-party path.
- Accessory role: yes. HAP-NodeJS (the engine under Homebridge) and Apple's open-source HomeKit ADK implement it. HomeKit has a Television accessory type: power, input selection, volume, playback keys, and the Apple TV Remote widget in Control Center attaches to it automatically. OpenStick exposes itself as a Television with one input per app.
- Risk: Apple's open HAP terms cover non-commercial use. Selling kits that include the integration is commercial. Homebridge is free software, not sold hardware, so its precedent is weaker than it looks. Options: (a) ship HAP only in the downloadable image, not as a feature of a sold kit; (b) apply to MFi; (c) rely on Matter for Apple Home instead. Decide before kits ship, with legal review.

**Matter**

- OpenStick exposes a Matter device with Basic Video Player, Media Playback, Keypad Input, On/Off and Content Launcher clusters, using the connectedhomeip SDK. One commissioning QR on the settings page targets Google Home, Alexa, SmartThings and Apple Home (as a Matter accessory, not a hub).
- Certification: shipping a Matter product needs CSA membership, a vendor ID, product certification, and a device attestation certificate per unit. Without it, ecosystems show an "uncertified device" warning, and some only accept test vendor IDs in developer mode **(verify per ecosystem)**. Options: certify through CSA (membership plus testing), ship as uncertified with documentation, or treat Matter as a DIY feature in v1.
- Ecosystem support for the Video Player device type is uneven; each hub may expose only On/Off or nothing. Test each before claiming support.
- Matter Casting rides on the same clusters; v2 once senders beyond Prime Video and Tubi exist.
- Thread border router: v2, needs a Thread radio no candidate board carries.

**Home Assistant**

- Native integration: mDNS discovery, media\_player entity with source list (apps), remote entity, power via CEC, plus a notify service that overlays a message on the TV. A custom component (HACS) ships first, then is submitted to HA core.
- Also a DLNA renderer so HA's existing DLNA integration works on day one with no code.

**Protocols the stick speaks on the LAN**

| Protocol | What it enables | v1 |
| --- | --- | --- |
| DIAL | Phone YouTube app finds the stick and launches its YouTube app with a session handoff | Yes, if the YouTube path supports it |
| DLNA/UPnP renderer | Any media server or phone app can push video to it | Yes |
| Spotify via librespot | Stick shows up as a target in the Spotify app | Image-only, off by default; librespot is unofficial and Spotify's developer terms likely bar it in commercial hardware **(verify)**; do not use the "Spotify Connect" name |
| mDNS/Bonjour | Discovery for everything above | Yes |
| Wake-on-LAN and CEC one-touch-play | A cast request turns the TV on and switches input, the Chromecast feature people love most | Yes on boards with working CEC |
| Bluetooth A2DP sink and source | Phone audio to the TV; TV audio to headphones | Yes |
| Bluetooth HID | Controllers, keyboards, remotes | Yes |
| Local REST/WebSocket API with token auth | Scripting, HA, third-party remotes | Yes |
| SSH | Off by default, toggle in settings | Yes |
| Signed OTA with rollback | Five-year support | Yes |
| Matter Casting receiver | Prime Video and Tubi senders today | v2 |
| Thread border router | Smart-home hub role | v2, needs a radio |

**Phone-free features**

A local web UI at openstick.local serves settings, a touch remote, a "paste a URL" receiver, and a photo/ambient source picker (local folder, Immich, Nextcloud, S3). There is no mobile app to install or maintain.

**Open items**

- [ ] Validate HAP Television accessory behavior with Control Center remote on iOS 26.
- [ ] Confirm which Matter video-player clusters Apple Home, Google Home, Alexa and SmartThings each honor.
- [ ] Decide the Matter certification path and the HAP commercial-use path.

## Website: 3D exploded views and streaming animations

The site is one Astro project: marketing pages with React Three Fiber islands for the 3D, Starlight for the build guide, deployed as static assets on Cloudflare Workers. The 3D bundle loads only on the pages that use it.

**Stack**

| Layer | Pick | Why |
| --- | --- | --- |
| Framework | [Astro](https://docs.astro.build/en/concepts/islands/) | Static HTML by default; `client:visible` defers the 3D island until it scrolls into view |
| Docs | [Starlight](https://starlight.astro.build/) | Same repo and build as the marketing site; search, dark mode, MDX, React components in docs pages |
| 3D | three.js + [React Three Fiber](https://r3f.docs.pmnd.rs/advanced/scaling-performance) + drei | Scriptable scene graph; `frameloop="demand"` so the GPU idles when the user is not scrolling |
| Scroll timeline | [GSAP ScrollTrigger](https://gsap.com/docs/v3/Plugins/ScrollTrigger/) | `scrub` + `pin` maps scroll position to a timeline; GSAP and plugins are now free for commercial use |
| Streaming diagrams | Inline SVG, `stroke-dashoffset` animation driven by GSAP | Tiny, pausable, versionable in git; Rive only if state machines are later needed |
| Model pipeline | KiCad + Onshape/Fusion, then [glTF Transform](https://gltf-transform.dev/) with meshopt and KTX2 | One GLB per part group, compressed |
| Hosting | [Cloudflare Workers static assets](https://docs.astro.build/en/guides/deploy/cloudflare/) | Cloudflare now points new projects to Workers rather than Pages |

Skipped: Spline (hosted editor, heavy runtime, poor fit for CAD-derived parts), Babylon.js (no advantage over R3F here), Lottie (loop-only export), Mintlify (proprietary). drei's `ScrollControls` is also out; it adds damping that fights ScrollTrigger pins.

**Exploded view: how it works**

1. The hero is a pinned section 500 to 900vh tall with one GSAP timeline, `scrub: 0.5`, `ease: none`.
2. The timeline tweens plain JS refs (camera position, look-at, one explode offset per part), never React state. The R3F `useFrame` loop reads the refs and applies them, so there are no re-renders.
3. Chapters in order: enclosure top lifts, heatsink, main board, expansion board, enclosure bottom, HDMI plug, remote. Each part has an offset along its own axis from assembled (0) to exploded (1). Scrolling back gives assembly for free.
4. Every part is a named node in the GLB (`enclosure_top`, `board`, `heatsink`, ...) so the timeline addresses it by name.
5. A config array maps scroll ranges to camera shots; pacing is tuned there, not in code. Theatre.js is the option if a visual timeline editor is wanted later.

The chapter list depends on the v1 board decision; build the pipeline against the v0 model and swap the GLB.

Reference implementation: [Codrops, cinematic 3D scroll with GSAP and R3F](https://tympanus.net/codrops/2025/11/19/how-to-build-cinematic-3d-scroll-experiences-with-gsap/).

**Streaming animations**

One SVG scene with five nodes (Phone, Router, OpenStick, Internet, TV) and a state enum: `idle`, `discover` (rings pulse from the phone), `handshake` (dashed line draws phone to stick), `fetch` (stick pulls from the Internet node), `playing` (HDMI line to the TV lights). This is the technically correct picture: for URL-based casting the stick fetches the stream itself and the phone is only the remote. Paths use `pathLength="1"` and animate `stroke-dashoffset` 1 to 0; packet dots ride `animateMotion` or GSAP MotionPath. The same scene scrubs on scroll in the explainer and auto-plays in the docs.

**Asset pipeline**

- Orange Pi does not publish a confirmed STEP for the Zero 2W. The Printables STEP that exists is for the Zero 2, a different board. Plan: model the 65 x 30 x 1.2 mm board from measured connector positions, or draw the custom carrier in KiCad and export GLB directly (KiCad exports STEP/GLB from the 3D viewer; Edge.Cuts must be closed).
- Enclosure, heatsink, HDMI plug and remote in Onshape or Fusion, exported as STEP, converted to glTF in Blender or FreeCAD, then `gltf-transform optimize` with meshopt. KTX2 or WebP only for the few textured parts. Join meshes that never move independently to cut draw calls.
- Budgets: hero GLB under 1 MB, whole scene under 4 MB, first 3D JS payload under 300 KB gzipped. A low-poly mobile variant or static poster on phones.

**Pages**

1. Home: hero exploded view, three-line pitch, "what it can't do" line, "build it" call to action, and "buy the kit" once distribution is decided.
2. How it works: the streaming animations, casting protocols, the honest streaming-services table fed by the playback lab.
3. Hardware: parts gallery with a `model-viewer` spin per part, BOM table with live links.
4. Build guide (Starlight): flash, assemble, pair remote, first boot, recovery. Same content as text, never only in the animation.
5. Downloads, community, blog, security advisories, privacy policy.

**Accessibility and performance**

- `prefers-reduced-motion` branches through `gsap.matchMedia()`: skip the pinned WebGL scene, show assembled and exploded stills rendered from the same GLB, a labeled parts list, and a play button. SVG diagrams get a pause control. Meets [WCAG 2.2 SC 2.3.3](https://www.w3.org/WAI/WCAG22/Understanding/animation-from-interactions.html).
- Poster image in the hero for LCP; canvas never gates it. DPR capped at 2 with drei `PerformanceMonitor` lowering it dynamically.
- Targets: LCP 2.5 s or better at p75 ([web.dev](https://web.dev/articles/lcp)); Lighthouse 90+ on pages without 3D, 80+ on the 3D page on mobile.

**Open items**

- [ ] Confirm current GSAP licensing covers ScrollTrigger and whether ScrollSmoother is still gated.
- [ ] Check orangepi.org downloads manually for a real Zero 2W STEP or DXF.
- [ ] Test the hero GLB on a mid-range phone before deciding on a mobile model variant.

## Distribution and support

Open question: kits, pre-built devices, flashed cards for a user-bought board, or files only. Each step up adds obligations:

| Model | Adds |
| --- | --- |
| Files only | Nothing beyond the open-source licences |
| Flashed card or software image sold | Consumer-law warranty on the item sold; HAP and trademark questions become commercial |
| Kit with board, case, remote, PSU | Product compliance (see Compliance), returns, warranty, fulfilment, payments |
| Pre-built device | All of the above plus final-product radio and safety certification |

Recommendation: files and images only through beta; decide on kits at the v1 GA gate.

## Compliance and regulatory

Required before anything is sold as a physical product:

- **EMC and radio:** FCC Part 15 (US) supplier's declaration for the finished device; CE (RED and EMC directives) and UKCA for Europe and the UK. Pre-certified Wi-Fi/BT modules reduce radio testing but do not exempt the finished product **(verify scope with a test lab)**.
- **Safety:** the wall adapter must carry its own UL/CE certification; buy a certified one, do not design one.
- **Materials and waste:** RoHS and REACH declarations from part suppliers; WEEE registration in EU countries where kits ship; battery rules for the remote's cells.
- **Trademarks:** do not use "AirPlay", "Miracast", "Spotify Connect", "Chromecast", "HomeKit" or "Matter" logos or certified marks without licence or certification. Describe compatibility in plain words ("works with iPhone screen mirroring") and have a lawyer review the wording.
- **Privacy:** a privacy policy for the website, the update server (which sees device IP addresses) and the opt-in crash endpoint; GDPR-compliant retention.

## Milestones

Five phases, each closed by a gate that is a test on hardware rather than a date. The v0 prototype starts on the Zero 2W already in hand; the board spike runs in parallel and decides the v1 reference before alpha work begins.

`v0 prototype → [gate 1] → Board spike → [gate 2] → v1 alpha → [gate 3] → Site and beta → [gate 4] → v1 GA`

| Phase | Scope | Exit gate |
| --- | --- | --- |
| v0 prototype | Debian-based image on the Zero 2W with patched HDMI kernel, Google Chrome kiosk, CEC test (informational), measured per-service resolution, default YouTube path chosen, QR Wi-Fi onboarding prototype | Netflix, Disney+, HBO Max play at 720p or better in Chrome on any test board; YouTube path decided |
| Board spike | CM5 stick, full RK3588S board, N100-class box: the eight spike exit criteria | Stick-vs-4K decision recorded; one v1 reference chosen; one remote chosen |
| v1 alpha | Launcher, `openstickd`, app manifest and store index, FCast, UxPlay, DIAL, DLNA, signed OTA with rollback and recovery, HomeKit Television, Matter video player, HA custom component, playback lab | Weekly automated playback and casting checks pass for 30 days; OTA fault-injection passes |
| Site and beta | Astro + Starlight site with the 3D exploded hero and streaming animations, enclosure CAD and STEP released, 50 beta builds by community testers, security disclosure process live | All beta-reported blockers closed; first-run success 90%; Lighthouse targets met |
| v1 GA | Signed release image, build guide, HA integration submitted upstream, distribution model decided; if selling kits, compliance done | Launch |

v2 candidates, not scheduled: Matter Casting receiver, Thread border router on a board with a radio, voice via open STT, custom carrier PCB with HDMI plug.

## Risks, open questions, legal

The risks that can kill v1 are the YouTube path, the Chrome-on-Linux path, and a streaming service blocking Linux browsers after launch. Each has an early test.

| Risk | Likelihood | Impact | Mitigation |
| --- | --- | --- | --- |
| YouTube blocks or breaks the chosen client path (TV interface, add-on API, attestation) | Medium-high | The most-used service degrades | Decide the path in v0; keep two paths working; desktop youtube.com in Chrome as the floor |
| Google drops or breaks Chrome for arm64 Linux, or its CDM | Medium | Web-app layer gone on ARM | x86 reference keeps the path; pin and roll back Chrome in the `vendor` partition |
| A major service blocks Linux browsers (Peacock already does) | Medium, ongoing | A "supported" tile goes dark | Weekly playback lab; tiles show ceilings; no service is promised in marketing copy |
| Netflix L3 ceiling drops from 720p to SD | Medium | Weakens a secondary use case | Position YouTube, self-hosted media and casting as primary |
| Chosen v1 board cannot deliver 4K YouTube | High for CM5, low for Intel | Headline 4K claim fails | Stick-vs-4K decision in the spike; only claim 4K YouTube on boards that pass |
| H618 HDMI and decode never reach mainline | High | v0 board becomes a dead end | Expected; v0 is a throwaway |
| Matter certification cost or ecosystem rejection | Medium | "Works with Google Home / Alexa" claim fails | Decide certification path before beta; ship HA and HAP regardless |
| HomeKit commercial use without MFi | Medium | Takedown request against sold kits | Image-only distribution, Matter as alternative, legal review |
| Trademark misuse (AirPlay, Miracast, Spotify, Chromecast) | Medium | Takedown or forced rename | Plain-language compatibility claims; legal review of all marketing copy |
| librespot breaks or Spotify enforces terms | Medium | Spotify target disappears | Ship off by default, unbranded, image-only |
| Google Cast expectations | High | Reviews say "it is not a Chromecast" | Site says so up front; DIAL covers the YouTube case; Netflix dropped casting anyway |
| Miracast Wi-Fi Direct unsupported on the radio | Medium | Windows and Android mirroring missing | Ship as experimental; AirPlay and FCast cover most users |
| Board supply (several candidates out of stock) | Medium | Builders cannot source the reference | Two-distributor rule; named second-source boards |
| Solo-founder bandwidth across hardware, OS, site and 3D | High | Schedule | Phase gates are tests, not dates; outsource the 3D hero |
| Founder unavailable during the five-year promise | Low per year, real over five | Updates stop; signing key lost | Continuity plan below |
| Trademark collision with the existing OpenStick modem project | Low | Search confusion | Brand as OpenStick.tv everywhere; GitHub org `opensticktv` |

**Continuity plan (the five-year promise)**

- The GitHub org has at least two owners before beta.
- The update-signing key has an escrowed backup held by a second trusted person or a legal escrow, with written conditions for use.
- If the project ends, the final update switches devices to community builds or a documented self-update channel, and the signing-key handover is announced publicly.
- All build infrastructure is reproducible from the public repos; nothing depends on a private server that cannot be replaced.

**Open questions**

- [ ] Stick form factor or 4K YouTube everywhere?
- [ ] Does any RK3588S board exist in a true stick form factor?
- [ ] Flutter or web for the launcher?
- [ ] Files only, flashed cards, kits, or pre-built devices?
- [ ] Matter: certify through CSA or ship uncertified?
- [ ] Which 3D artist or studio for the hero exploded-view model, if outsourced?

**Legal and licensing notes**

- Widevine CDM: distributed by Google inside official Chrome; OpenStick must not redistribute Chrome or the CDM in its image. The image installs Chrome from Google's repo on first run, after the user accepts Google's terms.
- Dolby Vision and Dolby Atmos decode are licensed; passthrough of bitstreams is fine, decode is not shipped.
- HDMI and HDCP: using the board's HDMI output is covered by the board vendor; OpenStick does not implement HDCP. Using the "HDMI" word mark in marketing is governed by HDMI Licensing **(verify)**.
- HomeKit: HAP is open for non-commercial use; commercial use requires MFi. See Ecosystem for options. OpenStick uses no Apple marks.
- AirPlay, Miracast, Spotify Connect: certified or licensed marks; describe compatibility without them.
- Matter: the Matter name and logo require CSA certification.
- Google Cast: no receiver licence available; OpenStick does not emulate a Cast receiver for stock senders and does not use the Chromecast name.
- Open-source licences: OS image and software Apache-2.0, hardware files CERN-OHL-S-2.0, site and docs content CC BY 4.0. UxPlay and Kodi are GPL and run as separate programs; the image ships their source offers.
- Naming: "OpenStick" is used by an unrelated modem-hacking project; no registered mark was found. Brand consistently as OpenStick.tv; consider a trademark filing before kits ship.
- Not legal advice; have a lawyer review HomeKit, Matter, trademark wording, Chrome install flow and compliance before anything is sold.

## Sources

Pages opened for this PRD, grouped by section. Statements marked **(verify)** in revision 2 are not yet backed by a source here.

**Chromecast and Google**

- [Google TV Streamer specs](https://store.google.com/product/google_tv_streamer_specs)
- [Google Chromecast spec table](https://support.google.com/chromecast/answer/3046409)
- [Google HDMI-CEC help](https://support.google.com/chromecast/answer/7199917)
- [Google TV Streamer launch post](https://blog.google/products/google-nest/google-tv-streamer/)
- [PCWorld Google TV Streamer review](https://www.pcworld.com/article/2494628/google-tv-streamer-review.html)
- [TechRadar on guest mode removal](https://www.techradar.com/televisions/streaming-devices/google-just-removed-one-of-chromecasts-most-useful-features)
- [Google Cast receiver authentication analysis](https://yingtongli.me/blog/2019/12/20/gcast-auth.html)
- [Android Authority on Netflix dropping casting](https://www.androidauthority.com/netflix-casting-chromecast-google-tv-streamer-3620784)

**DRM and streaming services**

- [Widevine DRM partner process](https://www.widevine.com/solutions/widevine-drm)
- [Khadas forum on Widevine L1](https://forum.khadas.com/t/widevine-l1-for-khadas-vim/885)
- [OMG Ubuntu on Chrome for arm64 Linux](https://www.omgubuntu.co.uk/2026/07/chrome-arm64-linux-available)
- [InputStream Adaptive Widevine ARM64 wiki](https://github.com/xbmc/inputstream.adaptive/wiki/Widevine-ARM64-support)
- [CastagnaIT Netflix add-on README](https://github.com/CastagnaIT/plugin.video.netflix)
- [CastagnaIT issue #1627](https://github.com/CastagnaIT/plugin.video.netflix/issues/1627)
- [Netflix on Linux browsers](https://help.netflix.com/en/node/30081)
- [Netflix on uncertified devices](https://help.netflix.com/en/node/124156)
- [HBO Max web resolution](https://help.hbomax.com/la-en/Answer/Detail/000002521)
- [Apple TV+ in a browser](https://support.apple.com/guide/tvplus/watch-in-a-web-browser-apdc0cb7ad64/web)
- [Prime Video on Linux, April 2026](https://en.linuxadictos.com/My-latest-failure-trying-to-play-Prime-Video-at-1080p-on-Linux.html)
- [Peacock blocking Linux, March 2026](https://ubuntu-mate.community/t/peacock-tv-hates-linux/31289)
- [Chromebook streaming caps, July 2026](https://www.starryhope.com/chromebooks/netflix-chromebook-streaming-caps/)

**Casting protocols**

- [FCast docs](https://docs.fcast.org) and [FCast GitHub](https://github.com/futo-org/fcast)
- [FCast receiver on Flathub](https://flathub.org/apps/org.fcast.Receiver)
- [UxPlay](https://github.com/FDH2/UxPlay)
- [MiracleCast](https://github.com/albfan/miraclecast)
- [TCL on Matter Casting](https://support.tcl.com/en_CA/what-is-matter-casting)
- [Matter Alpha on Matter Casting 1.4](https://www.matteralpha.com/news/amazon-to-update-matter-casting-to-1-4-spec)

**Hardware**

- [Orange Pi Zero 2W product page](http://www.orangepi.org/html/hardWare/computerAndMicrocontrollers/details/Orange-Pi-Zero-2W.html) and [wiki](http://www.orangepi.org/orangepiwiki/index.php/Orange_Pi_Zero_2W)
- [CNX Software on the Zero 2W](https://www.cnx-software.com/2023/09/09/orange-pi-zero-2w-raspberry-pi-zero-2w-alternative-4gb-ram/)
- [CNX Software H618 spec sheet](https://www.cnx-software.com/2022/07/30/allwinner-h618-processor-powers-android-12-tv-boxes/)
- [linux-sunxi mainlining status](https://linux-sunxi.org/Linux_mainlining_effort) and [Cedrus status](https://linux-sunxi.org/Sunxi-Cedrus)
- [H616 Cedrus patch series, May 2026](https://ratatoskr.run/linux-arm-kernel/2026/05/3545331/t)
- [H616 display patch NACK, September 2026](https://ratatoskr.run/lkml/2026/09/17516919/t)
- [LibreELEC Orange Pi Zero 2W thread](https://forum.libreelec.tv/thread/27650-orange-pi-zero-2w/)
- [Armbian Zero 3 thread with dw\_hdmi\_cec](https://forum.armbian.com/topic/29202-orange-pi-zero-3/page/3/)
- [Manjaro H618 hardware decode thread](https://forum.manjaro.org/t/support-for-orangepi-zero2-h616-ddr3-zero3-and-zero-2w-h618-lpddr4/154679)
- [AndroidPimp Zero 3W power and thermal](https://www.androidpimp.com/embedded/orange-pi-zero-3w-review/10/)
- [Makerfabs CM5 TV Stick Lite](https://liliputing.com/turn-a-raspberry-pi-cm5-into-a-tv-stick-with-this-carrier-board/)
- [Radxa ROCK 5C and 5C Lite](https://www.cnx-software.com/?p=130106)
- [Radxa Zero 3W at AmeriDroid](https://ameridroid.com/products/radxa-zero-3w)
- [MX3 air mouse](https://www.cnx-software.com/?p=42407)
- [Nordic forum on BLE remote voice on Linux](https://devzone.nordicsemi.com/f/nordic-q-a/39863/how-to-receive-voice-data-from-nordic-52832-remote-control-in-embeded-linux/154624)

**Website**

- [Astro islands](https://docs.astro.build/en/concepts/islands/) and [Cloudflare deploy guide](https://docs.astro.build/en/guides/deploy/cloudflare/)
- [Starlight](https://starlight.astro.build/)
- [GSAP ScrollTrigger](https://gsap.com/docs/v3/Plugins/ScrollTrigger/)
- [GSAP forum: ScrollTrigger vs drei ScrollControls](https://gsap.com/community/forums/topic/40114-scrolltrigger-pin-and-dreis-scrollcontrols-dont-play-well-together/)
- [Codrops: cinematic 3D scroll with GSAP and R3F](https://tympanus.net/codrops/2025/11/19/how-to-build-cinematic-3d-scroll-experiences-with-gsap/)
- [Codrops: Theatre.js camera fly-through](https://tympanus.net/codrops/2023/02/14/animate-a-camera-fly-through-on-scroll-using-theatre-js-and-react-three-fiber/)
- [R3F scaling performance](https://r3f.docs.pmnd.rs/advanced/scaling-performance)
- [glTF Transform](https://gltf-transform.dev/) and [gltfpack](https://github.com/zeux/meshoptimizer/blob/master/gltf/README.md)
- [KiCad 3D export](https://docs.kicad.org/9.0/en/pcbnew/pcbnew.html)
- [MDN stroke-dashoffset](https://developer.mozilla.org/en-US/docs/Web/CSS/stroke-dashoffset)
- [WCAG 2.2 SC 2.3.3](https://www.w3.org/WAI/WCAG22/Understanding/animation-from-interactions.html)
- [web.dev LCP](https://web.dev/articles/lcp)
- [Flipper docs](https://docs.flipper.net/) and [Framework](https://frame.work/) as structural references

**Naming**

- [OpenStick modem project notes](https://wiki2.realraum.at/memberpages:julianm:projects:openstick)
