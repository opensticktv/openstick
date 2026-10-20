# OpenStick.tv PRD

Oct 20, 2026 · @Chris

## Summary

OpenStick.tv is an open-hardware, open-software HDMI stick that matches a Google TV Streamer on casting, CEC, 4K output, smart-home control and longevity, and is honest about the one thing it cannot do: play Netflix, Disney+, Max, Prime or Apple TV+ above SD to 720p. That ceiling is set by Widevine L1 and per-service certification, which no open device can obtain, and it is the same ceiling every Linux box, Raspberry Pi and uncertified Android stick already lives with.

Three findings from the research change the plan:

1. The Orange Pi Zero 2W is a v0 prototyping board only. Its HDMI output is not in mainline Linux, its hardware decoder is not usable on the vendor kernel, it has no AV1 (so no 4K YouTube), and its price has roughly doubled since launch. v1 moves to a Pi CM5 stick carrier or an RK3588-family board after a two-week spike.
2. The DRM story is software Widevine L3 through official Google Chrome for arm64 Linux, which Google began supporting formally in August 2026. Expect 720p on Netflix and Disney+, 720p on Max, SD on Prime, and unknown on Apple TV+. The Kodi Netflix add-on is dead. The Android TV route gives the same SD ceiling with a worse stack.
3. Google Cast is not reachable: stock phones reject uncertified receivers and no licence is offered. The open substitutes are FCast, AirPlay (UxPlay), Miracast, DIAL and DLNA, and Netflix itself dropped phone casting for most devices in December 2025, which shrinks the gap.

The differentiators that remain are real: guest casting Google removed, Apple Home and Matter control of the TV, Home Assistant integration, no telemetry, a local API, five years of signed updates, and a website that teaches the hardware through interactive 3D exploded views. Target price is under $89 built with the v0 BOM; v1 hardware will land higher.

The PRD is organized as goals, the Chromecast parity matrix, streaming and DRM, hardware, software, ecosystem integrations, website, milestones, and risks.

## Goals and non-goals

OpenStick ships a sub-$50 open-hardware, open-software HDMI stick that does what a Chromecast does for the things an open device can legally do, and says plainly what it cannot.

**Goals (v1)**

- Plug-and-play HDMI stick: TV-powered where possible, 4K60 output, HDMI-CEC so the TV remote works on day one.
- Casting receiver for phones and laptops: AirPlay, Miracast, FCast, and DIAL-style app launch, with Google Cast handled through the best open path available.
- A 10-foot UI with a launcher, app model, settings, and OTA updates, built on a mainline-Linux image that anyone can rebuild from source.
- Streaming apps for the major services at the best quality open hardware is permitted to deliver, with the resolution ceiling shown honestly in the app tile.
- Fully published hardware: BOM, enclosure CAD, carrier/expansion board files, and a build guide a hobbyist can follow in an afternoon.
- A website that teaches the product through interactive 3D exploded views and animated streaming diagrams, and doubles as the build guide.

**Non-goals (v1)**

- Matching Chromecast on DRM-gated 4K from Netflix, Disney+, Max or Apple TV+. That needs Widevine L1 plus per-service certification, which is not available to an open device. Section 4 covers what is reachable.
- Google Assistant, Google TV recommendations, or any feature that depends on Google Play Services.
- Custom silicon or a custom SoC carrier board. v0 and v1 use an off-the-shelf board; a custom carrier is a v2 question.
- Being a general-purpose desktop. The image boots to the 10-foot UI; a shell is available but not the product.
- A mobile companion app beyond what casting protocols already provide.

## Chromecast parity matrix

Of the 18 things a Google TV Streamer does, OpenStick can match 11 outright, approximate 4, and cannot do 3 (Google Cast, Assistant, DRM-gated 4K). The benchmark is the [Google TV Streamer (4K, 2024)](https://store.google.com/product/google_tv_streamer_specs), $99.99 at launch, with the 2020 Chromecast with Google TV as the floor.

| Capability | Google TV Streamer | OpenStick v1 | How |
| --- | --- | --- | --- |
| 4K60 output | Yes | Yes | Board HDMI 2.0; H618 only on vendor kernel, RK3588/CM5 on mainline |
| HDR10 / HDR10+ / Dolby Vision | All four plus HLG | HDR10 partial, no DV | Dolby Vision is licensed; HDR10 metadata passthrough unproven on H618 under Linux, works on RK3588 and Pi 5 |
| Dolby Atmos / DD+ passthrough | Yes | Passthrough only | Kodi passes bitstreams; no Dolby decode license needed for passthrough |
| Google Cast receiver (video, audio) | Yes | No | Cast receiver requires a Google license; see Software for the open substitutes |
| Screen and tab mirroring | From Android and Chrome | Yes | Miracast (Android, Windows) via MiracleCast; AirPlay mirroring via UxPlay; Chrome tab via WebRTC page |
| Launcher / home with recommendations | Google TV | Launcher, no recommendations | Own 10-foot UI; no Google content graph |
| Voice remote and Assistant | Yes, plus Gemini | No (v1), partial (v2) | BLE remote mics use vendor GATT audio with no Linux receiver; v2 option is phone-as-mic or USB mic with an open STT model |
| HDMI-CEC: TV power on, input switch on cast | Yes, wall power required | Yes if CEC validates | libCEC; Google itself requires wall power for this |
| Ethernet | Built-in 1 Gbps | 100M via expansion board, 1G on v1 boards |  |
| Bluetooth remote, game controllers, headphones | BT 5.1 | Yes | BlueZ; BT 5.0 or 5.4 depending on board |
| Wi-Fi | 802.11ac dual band | Yes, Wi-Fi 5 or 6 by board |  |
| Ambient mode / screensaver with photos | Google Photos | Yes, any source | Local, S3, Immich, or Nextcloud photo sources |
| Multi-user and kids profiles | Yes | Yes, simpler | Profile switcher with PIN; parental controls per app |
| Find my remote | Button and voice | Yes with BLE remote | Buzzer in remote; most BLE remotes lack one, so v1 ships a remote that has it or skips the feature |
| Guest mode | Removed by Google in 2023 | Yes | Guest cast without Wi-Fi credentials is a differentiator Google dropped |
| Matter and Thread border router | Yes | Matter device v1, Thread v2 | Exposes itself as a Matter TV endpoint; Thread needs a radio, which no candidate board has |
| Five years of OS updates | Yes | Yes by design | Signed OTA with A/B rollback; open source outlives the vendor |
| Streaming apps at 4K with DRM | Yes, certified | No at 4K | See Streaming services and DRM |
| USB for storage or wired accessories | None, needs a hub | Yes | USB-A on the expansion board or carrier |
| Price | $99.99 | Target under $89 built, under $50 board-only | DRAM pricing in 2026 makes under $50 all-in unrealistic; see Hardware |

What Google gets from its chip that the v0 board does not: AV1 decode (YouTube 4K is AV1-first), a certified DRM TEE, and Dolby Vision. The first two drive the v1 board choice.

Sources: [Google spec table](https://support.google.com/chromecast/answer/3046409), [Google CEC page](https://support.google.com/chromecast/answer/7199917), [Streamer launch post](https://blog.google/products/google-nest/google-tv-streamer/), [PCWorld review](https://www.pcworld.com/article/2494628/google-tv-streamer-review.html), [TechRadar on guest mode removal](https://www.techradar.com/televisions/streaming-devices/google-just-removed-one-of-chromecasts-most-useful-features).

## Streaming services and DRM

OpenStick gets software Widevine L3 and nothing higher, so the major subscription services top out at SD to 720p and 4K is off the table on any open board. The product promise is YouTube, self-hosted media, open casting, and "your subscriptions in a browser at reduced quality."

**Why: the three locks**

1. Widevine L1 needs a secure bootloader, a factory-provisioned device-unique keybox, and decryption inside a TEE, plus a signed agreement and robustness review with Google ([Widevine](https://www.widevine.com/solutions/widevine-drm), [Khadas thread](https://forum.khadas.com/t/widevine-l1-for-khadas-vim/885)). No open-hardware project has obtained it. Custom ROMs that keep L1 (GrapheneOS on Pixel) ride the phone vendor's existing keybox, which does not transfer to a hobby board.
2. Each service then certifies devices separately. Netflix sends uncertified-device users to a Play Protect certified device ([Netflix](https://help.netflix.com/en/node/124156)); uncertified Android boxes get SD at best and are sometimes blocked outright.
3. Google Cast senders verify a Google-issued certificate on the receiver, so a third-party Cast receiver is rejected by stock phones ([gcast-auth](https://yingtongli.me/blog/2019/12/20/gcast-auth.html)). No public receiver licence exists. Netflix also removed phone casting for most devices in December 2025 ([Android Authority](https://www.androidauthority.com/netflix-casting-chromecast-google-tv-streamer-3620784)), so Cast parity matters less than it did.

**What works on Linux arm64 in 2026**

Google ships official Chrome for arm64 Linux with a native Widevine L3 module, formally supported since August 2026 ([OMG Ubuntu](https://www.omgubuntu.co.uk/2026/07/chrome-arm64-linux-available)). That is the path: a kiosk Chromium with per-service web apps. Kodi's InputStream Adaptive can load the same CDM, but the CastagnaIT Netflix add-on is suspended and effectively broken since Netflix changed its licence flow ([issue #1627](https://github.com/CastagnaIT/plugin.video.netflix/issues/1627)).

| Service | Path on OpenStick | What you get | Confidence |
| --- | --- | --- | --- |
| YouTube | Native app (Kodi add-on or Chromium) | Up to 4K where the board decodes it; 4K is AV1-first, so no AV1 hardware means 1080p on H618 | High |
| Jellyfin, Plex, Emby, Kodi library | Native | Full quality, hardware decode | High |
| Netflix | Chromium web app, L3 | 720p likely, 1080p claimed by [Netflix's help page](https://help.netflix.com/en/node/30081), which also says Linux is unsupported | Medium; no authoritative 2026 ARM test |
| Disney+ | Chromium, L3 | 720p, no 4K on any computer browser | Medium; intermittent 403 reports |
| HBO Max | Chromium | 720p is the [cap for the website](https://help.hbomax.com/la-en/Answer/Detail/000002521) on any device | Medium |
| Prime Video | Chromium, L3 | SD (480p to 540p); Amazon requires L1 for anything higher | High |
| Apple TV+ | tv.apple.com in Chromium | Undocumented; no non-browser path exists; AirPlay from the Apple TV app passes audio only, UxPlay cannot decrypt Apple video DRM | Low |
| Hulu, Paramount+ | Chromium | Unverified | Low |
| Peacock | Chromium | Actively blocking Linux browsers as of March 2026 | Blocked |

**The Android TV option, and why not**

Orange Pi ships an Android 12 TV image, but it is not Play Protect certified and carries no L1 keybox. Sideloaded Netflix on an uncertified box gets SD or nothing. No LineageOS or Android TV community build exists for the H618. Android would trade an open stack for an unmaintained vendor image and the same SD ceiling. Not worth it.

**Product rules**

- Every service tile shows its real ceiling ("Netflix, up to 720p") and the setup page says why, in two sentences, with a link to this reasoning.
- No service is listed as "supported" until a weekly automated playback check passes for 30 days; Peacock-style blocks happen without notice.
- The web-app layer uses a maintained Chromium kiosk with the official arm64 CDM, auto-updated, not an extracted ChromeOS blob.
- If a certified-DRM tier is ever wanted, it is a separate product on a vendor-certified Android TV SoC and is out of scope for OpenStick's open image.

**Open items**

- [ ] Run Netflix, Disney+, Max, Prime and Apple TV+ in official Chrome arm64 on the v0 board and record the actual resolution each delivers.
- [ ] Confirm the arm64 CDM runtime requirements (glibc TLS patch, LSE atomics) against the chosen image ([ISA wiki](https://github.com/xbmc/inputstream.adaptive/wiki/Widevine-ARM64-support)).

## Hardware

The Orange Pi Zero 2W is a v0 prototyping board, not the v1 reference: its HDMI output is not in mainline Linux, hardware video decode on Linux is unfinished, and the Allwinner H618 has no AV1 decoder. v1 moves to an RK3588-family or Pi CM5 board, at roughly 2 to 3 times the board cost.

**Board decision**

| Board | SoC | RAM | HDMI out | Decode on Linux | Wi-Fi / BT | Price found (Oct 2026) | Verdict |
| --- | --- | --- | --- | --- | --- | --- | --- |
| [Orange Pi Zero 2W](http://www.orangepi.org/html/hardWare/computerAndMicrocontrollers/details/Orange-Pi-Zero-2W.html) | Allwinner H618, 4x A53 1.5 GHz | 1 to 4 GB | 4K60, mini HDMI | HEVC 4K30 on vendor Android only; mainline Cedrus for H616 posted May 2026, not merged; no AV1 | Wi-Fi 5 / BT 5.0 | $28 (1 GB) to $96 (4 GB) on AliExpress, up from $13 to $24 at launch | v0 prototype only |
| [Banana Pi BPI-M4 Zero](https://ameridroid.com/products/banana-pi-m4-zero) | H618 | 2 to 4 GB | 4K60, mini HDMI | Same as above, adds 8 GB eMMC | Wi-Fi 5 / BT 4.2 | $65, out of stock | No advantage |
| [Radxa Zero 3W](https://ameridroid.com/products/radxa-zero-3w) | RK3566 | 1 to 8 GB | 1080p60 on the spec sheet, micro HDMI | Good Rockchip VPU support | Wi-Fi 6 / BT 5.4 | $42, sold out | Fails the 4K requirement |
| [Radxa ROCK 5C Lite](https://www.cnx-software.com/?p=130106) | RK3582 (cut-down RK3588S) | 1 to 16 GB | 4K-class HDMI, full-size port | RK3588 VPU: HEVC/VP9/AV1 hardware decode, mature in mainline | Wi-Fi 6 / BT 5.4 | $34 (1 GB, out of stock), $52 (4 GB) | v1 candidate, credit-card size not stick size |
| Pi CM5 + [Makerfabs TV Stick Lite](https://liliputing.com/turn-a-raspberry-pi-cm5-into-a-tv-stick-with-this-carrier-board/) | BCM2712 | 2 to 16 GB | HDMI plug built into the carrier | HEVC 4K60 hardware, no AV1 or H.264 hardware decode | Wi-Fi 5 / BT 5.0 | $35 carrier + $68 CM5 | v1 candidate, true stick form, mainline Linux |
| [Raspberry Pi Zero 2 W](https://www.raspberrypi.com/products/raspberry-pi-zero-2-w/) | BCM2710 | 512 MB | 1080p | 1080p30 | Wi-Fi 4 / BT 4.2 | $15 list, out of stock | Fails 4K |

Recommendation: prototype the software on the Zero 2W now because it is in hand and cheap, and run a two-week hardware spike on the CM5 stick carrier and the ROCK 5C Lite before committing v1. The CM5 carrier is the only option with a real HDMI plug and no adapter. Open question: whether an RK3588S board exists in a true stick form factor; none was found.

**v0 BOM (Orange Pi Zero 2W)**

| Part | Spec | Price (USD) | Note |
| --- | --- | --- | --- |
| Orange Pi Zero 2W, 2 GB | H618, LPDDR4, Wi-Fi 5, BT 5.0, 16 MB SPI flash, microSD | about 45 | 1 GB is enough for Kodi; 2 GB for a browser-based app layer; check Amazon US by hand, it blocks price scraping |
| [24-pin expansion board](https://www.cnx-software.com/2023/09/09/orange-pi-zero-2w-raspberry-pi-zero-2w-alternative-4gb-ram/) | 2x USB-A, 100M Ethernet, IR receiver, 3.5 mm audio, power and user buttons | 5 | Required for IR and USB-A |
| microSD, 32 GB A1 | Boot and app storage | 6 | No eMMC on this board |
| Mini HDMI male to HDMI male adapter | Short rigid adapter forming the plug | 3 | Verify gender before buying; most sold adapters are the wrong way round |
| Wall adapter, 5 V 2 A USB-C | Board is rated 5 V 2 A, about 10 W peak | 6 | A TV USB port (2.5 to 4.5 W) is not reliable; Google requires wall power for CEC on Chromecast too |
| Heatsink, 20 mm with thermal pad | Passive | 2 | Orange Pi calls it optional; sibling Zero 3W hit 63 C at 60 s all-core; no sustained 4K data exists, so include it |
| Enclosure | 3D-printed two-part shell | 2 in filament | No existing stick case for this board; existing cases leave the port open for a cable. Custom design required |
| BLE or 2.4 GHz remote | See below | 9 to 16 |  |
| Total |  | 78 to 85 | Was estimated at 35 to 45 before the DRAM price rise and the wall adapter |

**HDMI-CEC**

Unproven on H618. The SoC has a DesignWare HDMI TX with a hardware CEC block, and `dw_hdmi_cec` loaded on a patched Armbian 6.6 kernel, but no one has reported `cec-client` or libCEC working on H616/H618 ([linux-sunxi status](https://linux-sunxi.org/Linux_mainlining_effort) shows "?"). Validate on hardware in week one. Fallback is `cec-gpio` if the pin is exposed. The CM5 route has CEC working today.

**Remote**

1. TV remote via CEC: zero cost if CEC works.
2. [MX3-class 2.4 GHz air mouse](https://www.cnx-software.com/?p=42407), $12 to $16: USB HID, works on any Linux, needs the expansion board's USB-A.
3. G10S Pro-class BLE remote, about $9: BLE HID for keys works through BlueZ. Voice does not: remote microphones use vendor-specific GATT audio, and no Linux receiver exists. Voice in v1 means phone-as-remote or a USB mic, not the remote.
4. IR via the expansion board and rc-core/LIRC: works for any universal remote.

**Decode and HDR facts for the H618** ([CNX spec sheet](https://www.cnx-software.com/2022/07/30/allwinner-h618-processor-powers-android-12-tv-boxes/)): HEVC Main10 and VP9 to 6Kp30, H.264 to 4Kp30, no AV1, HDMI 2.0 with HDR10 and HDCP 2.2 in silicon. On Linux: 1080p50 YouTube at 20% CPU with ffmpeg v4l2-request on Manjaro; HDR metadata has never been shown passing to the TV on Linux; the community LibreELEC port had HEVC artifacts and no HDMI audio ([LibreELEC thread](https://forum.libreelec.tv/thread/27650-orange-pi-zero-2w/)). Vendor Android 13 plays 4K30 with hardware decode.

**Open items**

- [ ] Order a CM5 + Makerfabs TV Stick Lite and a ROCK 5C Lite for the v1 spike.
- [ ] Test CEC on the Zero 2W with `cec-ctl` on the Armbian patched kernel.
- [ ] Measure sustained 4K decode temperature in a closed enclosure with and without the heatsink.
- [ ] Confirm the Orange Pi official AliExpress store price per RAM tier and the Amazon US price.

## Software platform

OpenStick OS is a signed, A/B-updatable Linux image built with Buildroot or Yocto on a mainline kernel, booting straight into a 10-foot launcher that hosts three app types: native (Kodi, Jellyfin), web (Chromium kiosk with the official arm64 Widevine CDM), and casting receivers.

**Layers**

| Layer | Choice | Note |
| --- | --- | --- |
| Build | Buildroot (v0), evaluate Yocto for v1 | Reproducible image from one repo; LibreELEC's build system is the model |
| Kernel | Mainline plus the minimum patch set the board needs | On H618 this means out-of-tree HDMI and Cedrus patches until they merge; on CM5/RK3588 it is close to clean mainline |
| Display | DRM/KMS, Wayland (wlroots-based compositor such as cage) | No X11; the compositor owns the HDMI output and hands it to whichever app is in front |
| Media | GStreamer with v4l2-request hardware decode; Kodi for the library | ffmpeg v4l2-request already does 1080p50 at 20% CPU on H618 |
| Launcher | Own 10-foot UI, Flutter or web (Chromium) | Web keeps one UI stack for launcher, settings and web apps; Flutter is leaner on RAM |
| Web apps | Chromium kiosk, one isolated profile per service, official Google Chrome arm64 CDM | Service tiles declare their resolution ceiling |
| Casting | FCast receiver (Flathub aarch64, v3.0.4), UxPlay for AirPlay, MiracleCast for Miracast, DIAL server, DLNA renderer | FCast includes a Google Cast receiver disabled by default; stock senders reject it, so it stays off |
| Updates | Signed OTA, A/B partitions, automatic rollback, RAUC or Mender | Five-year support promise depends on this |
| Remote input | libCEC, BlueZ HID, rc-core for IR, local web remote | Phone-as-remote via the local web UI needs no app |
| Local API | REST plus WebSocket over mDNS, token auth | Feeds Home Assistant and the Matter bridge |

**Casting receivers in detail**

- FCast ([docs](https://docs.fcast.org), [GitHub](https://github.com/futo-org/fcast)): open MIT protocol, DASH/HLS/mp4, mDNS discovery, HDR10 metadata passthrough since 3.0.3. Senders: Grayjay, FUTO desktop and terminal senders, and any browser with the FCast extension. No mainstream streaming app supports it. OpenStick should ship the receiver and promote Grayjay as the companion.
- AirPlay via [UxPlay](https://github.com/FDH2/UxPlay): GPL-3, mirroring and audio, YouTube from the iOS app with `-hls`, built-in mDNS since 1.74. No DRM video, no browser AirPlay.
- Miracast via [MiracleCast](https://github.com/albfan/miraclecast): needs Wi-Fi Direct on the board's radio, which is unverified on the Zero 2W, and conflicts with NetworkManager. Ship as experimental.
- DIAL: a small HTTP server that lets the phone's YouTube app find OpenStick and launch its YouTube app with a session handoff. This is how YouTube "casting" works without Google Cast.
- Matter Casting: Prime Video and Tubi are the only confirmed senders, receivers are Fire TV and Echo Show, and a Linux example TV app exists in connectedhomeip. v2 candidate, not v1.
- Google Cast: not possible for stock senders. Rooted-sender workarounds are not a product.

**App model**

Apps are OCI-style bundles (Flatpak on the image) with a manifest: name, icon, launch command, input mode (pointer, d-pad, keyboard), resolution ceiling, and required permissions (network, USB, mic). The launcher reads the manifest; the store is a static JSON index on openstick.tv plus any user-added repo URL.

**Security baseline**

Read-only root, verified boot where the board supports it, apps sandboxed (Flatpak portals), SSH off by default with a toggle in settings, no telemetry, update-server signing key in a hardware token.

**Open items**

- [ ] Pick Flutter vs web for the launcher after a RAM and cold-start test on 1 GB.
- [ ] Confirm Wi-Fi Direct support on the v0 and v1 radios for Miracast.
- [ ] Decide RAUC vs Mender for OTA.

## Ecosystem integrations

OpenStick appears as a TV accessory in Apple Home, Google Home, Alexa, SmartThings and Home Assistant through two protocols (HomeKit Accessory Protocol and Matter) and one local API, so a user can say "turn on the living room TV" in whatever assistant they already own. It cannot be an Apple Home hub.

**Apple Home**

- Hub role: no. Only Apple TV and HomePod can act as the home hub that runs automations and gives remote access. There is no third-party path.
- Accessory role: yes. HAP is open for non-commercial use, and HAP-NodeJS (the engine under Homebridge) and Apple's open-source HomeKit ADK implement it. HomeKit has a Television accessory type: power, input selection, volume, playback keys, and the Apple TV Remote widget in Control Center attaches to it automatically. OpenStick exposes itself as a Television with one input per app.
- Risk: commercial distribution with the HomeKit badge requires MFi. Homebridge has shipped without it for years. Decision: ship it, no badge, documented as community integration.

**Matter**

- OpenStick exposes a Matter device with Basic Video Player, Media Playback, Keypad Input, On/Off and Content Launcher clusters, using the connectedhomeip SDK. One commissioning QR on the settings page covers Google Home, Alexa, SmartThings and Apple Home (as a Matter accessory, not a hub).
- Matter Casting rides on the same clusters; v2 once senders beyond Prime Video and Tubi exist.
- Thread border router: v2, needs a Thread radio no candidate board carries.

**Home Assistant**

- Native integration: mDNS discovery, media\_player entity with source list (apps), remote entity, power via CEC, plus a notify service that overlays a message on the TV. Submitted to HA core; a custom component (HACS) ships first.
- Also a DLNA renderer so HA's existing DLNA integration works on day one with no code.

**Protocols the stick speaks on the LAN**

| Protocol | What it enables | v1 |
| --- | --- | --- |
| DIAL | Phone YouTube app finds the stick and launches its YouTube app with a session handoff | Yes |
| DLNA/UPnP renderer | Any media server or phone app can push video to it | Yes |
| Spotify Connect via librespot | Stick shows up as a target in the Spotify app | Yes |
| mDNS/Bonjour | Discovery for everything above | Yes |
| Wake-on-LAN and CEC one-touch-play | A cast request turns the TV on and switches input, the Chromecast feature people love most | Yes if CEC validates |
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
- [ ] Confirm which clusters Apple Home honors for a Matter video player (Apple's Matter support is narrower than Google's).

## Website: 3D exploded views and streaming animations

The site is one Astro project: marketing pages with React Three Fiber islands for the 3D, Starlight for the build guide, deployed as static assets on Cloudflare Workers. The 3D bundle loads only on the pages that use it.

**Stack**

| Layer | Pick | Why |
| --- | --- | --- |
| Framework | [Astro 7](https://docs.astro.build/en/concepts/islands/) | Static HTML by default; `client:visible` defers the 3D island until it scrolls into view |
| Docs | [Starlight](https://starlight.astro.build/) | Same repo and build as the marketing site; search, dark mode, MDX, React components in docs pages |
| 3D | three.js + [React Three Fiber](https://r3f.docs.pmnd.rs/advanced/scaling-performance) + drei | Scriptable scene graph; `frameloop="demand"` so the GPU idles when the user is not scrolling |
| Scroll timeline | [GSAP ScrollTrigger](https://gsap.com/docs/v3/Plugins/ScrollTrigger/) | `scrub` + `pin` maps scroll position to a timeline; GSAP and plugins are now free for commercial use |
| Streaming diagrams | Inline SVG, `stroke-dashoffset` animation driven by GSAP | Tiny, pausable, versionable in git; Rive only if state machines are later needed |
| Model pipeline | KiCad + Onshape/Fusion, then [glTF Transform](https://gltf-transform.dev/) with meshopt and KTX2 | One GLB per part group, compressed |
| Hosting | [Cloudflare Workers static assets](https://docs.astro.build/en/guides/deploy/cloudflare/) | Cloudflare now points new projects to Workers rather than Pages |

Skipped: Spline (hosted editor, heavy runtime, poor fit for CAD-derived parts), Babylon.js (no advantage over R3F here), Lottie (loop-only export), Mintlify (proprietary, $450+/month beyond Starter). drei's `ScrollControls` is also out; it adds damping that fights ScrollTrigger pins.

**Exploded view: how it works**

1. The hero is a pinned section 500 to 900vh tall with one GSAP timeline, `scrub: 0.5`, `ease: none`.
2. The timeline tweens plain JS refs (camera position, look-at, one explode offset per part), never React state. The R3F `useFrame` loop reads the refs and applies them, so there are no re-renders.
3. Chapters in order: enclosure top lifts, heatsink, main board, expansion board, enclosure bottom, HDMI plug, remote. Each part has an offset along its own axis from assembled (0) to exploded (1). Scrolling back gives assembly for free.
4. Every part is a named node in the GLB (`enclosure_top`, `board`, `heatsink`, ...) so the timeline addresses it by name.
5. A config array maps scroll ranges to camera shots; pacing is tuned there, not in code. Theatre.js is the option if a visual timeline editor is wanted later.

Reference implementation: [Codrops, cinematic 3D scroll with GSAP and R3F](https://tympanus.net/codrops/2025/11/19/how-to-build-cinematic-3d-scroll-experiences-with-gsap/).

**Streaming animations**

One SVG scene with five nodes (Phone, Router, OpenStick, Internet, TV) and a state enum: `idle`, `discover` (rings pulse from the phone), `handshake` (dashed line draws phone to stick), `fetch` (stick pulls from the Internet node), `playing` (HDMI line to the TV lights). This is the technically correct picture: for URL-based casting the stick fetches the stream itself and the phone is only the remote. Paths use `pathLength="1"` and animate `stroke-dashoffset` 1 to 0; packet dots ride `animateMotion` or GSAP MotionPath. The same scene scrubs on scroll in the explainer and auto-plays in the docs.

**Asset pipeline**

- Orange Pi does not publish a confirmed STEP for the Zero 2W. The Printables STEP that exists is for the Zero 2, a different board. Plan: model the 65 x 30 x 1.2 mm board from measured connector positions, or draw the custom carrier in KiCad and export GLB directly (KiCad exports STEP/GLB from the 3D viewer; Edge.Cuts must be closed).
- Enclosure, heatsink, HDMI plug and remote in Onshape or Fusion, exported as STEP, converted to glTF in Blender or FreeCAD, then `gltf-transform optimize` with meshopt. KTX2 or WebP only for the few textured parts. Join meshes that never move independently to cut draw calls.
- Budgets: hero GLB under 1 MB, whole scene under 4 MB, first 3D JS payload under 300 KB gzipped. A low-poly mobile variant or static poster on phones.

**Pages**

1. Home: hero exploded view, three-line pitch, price, "build it" and "buy the kit" calls to action.
2. How it works: the streaming animations, casting protocols, the honest streaming-services table.
3. Hardware: parts gallery with a `model-viewer` spin per part, BOM table with live links.
4. Build guide (Starlight): flash, assemble, pair remote, first boot. Same content as text, never only in the animation.
5. Downloads, community, blog.

**Accessibility and performance**

- `prefers-reduced-motion` branches through `gsap.matchMedia()`: skip the pinned WebGL scene, show assembled and exploded stills rendered from the same GLB, a labeled parts list, and a play button. SVG diagrams get a pause control. Meets [WCAG 2.2 SC 2.3.3](https://www.w3.org/WAI/WCAG22/Understanding/animation-from-interactions.html).
- Poster image in the hero for LCP; canvas never gates it. DPR capped at 2 with drei `PerformanceMonitor` lowering it dynamically.
- Targets: LCP 2.5 s or better at p75 ([web.dev](https://web.dev/articles/lcp)); Lighthouse 90+ on pages without 3D, 80+ on the 3D page on mobile.

**Open items**

- [ ] Confirm current GSAP licensing covers ScrollTrigger and whether ScrollSmoother is still gated.
- [ ] Check orangepi.org downloads manually for a real Zero 2W STEP or DXF.
- [ ] Test the hero GLB on a mid-range phone before deciding on a mobile model variant.

## Milestones

Five phases, each closed by a gate that is a test on hardware rather than a date. The v0 prototype starts on the Zero 2W already in hand; the board spike runs in parallel and decides the v1 reference before alpha work begins.

&#91;embedded content: roadmap · 5 phases, 4 gates\]

Each diamond is the exit test for the phase to its left; the first two gates are the ones the research says are most likely to fail (CEC on H618 and the v1 board cost).

| Phase | Scope | Exit gate |
| --- | --- | --- |
| v0 prototype | Buildroot image on the Zero 2W with patched HDMI kernel, Chromium kiosk with official arm64 CDM, CEC test, measured per-service resolution | CEC works; Netflix, Disney+, Max play at 720p or better in Chromium |
| Board spike | CM5 + Makerfabs TV Stick Lite and ROCK 5C Lite: 4K60 HEVC, AV1, HDR10 metadata out, CEC, sustained thermal in a closed shell | One board chosen; built BOM under $89 or the price target revised in writing |
| v1 alpha | Launcher, app manifest and store index, FCast, UxPlay, DIAL, DLNA, librespot, signed OTA with rollback, HomeKit Television, Matter video player, HA custom component | Weekly automated playback and casting checks pass for 30 days |
| Site and beta | Astro + Starlight site with the 3D exploded hero and streaming animations, enclosure CAD and STEP released, 50 beta kits to community testers | All beta-reported blockers closed; Lighthouse targets met |
| v1 GA | Signed release image, build guide, kit on sale, HA integration submitted upstream | Launch |

v2 candidates, not scheduled: Matter Casting receiver, Thread border router on a board with a radio, voice via open STT, custom carrier PCB with HDMI plug.

## Risks, open questions, legal

The two risks that can kill v1 are the H618's Linux display and decode support and a streaming service blocking Linux browsers after launch. Both have a named mitigation and an early test.

| Risk | Likelihood | Impact | Mitigation |
| --- | --- | --- | --- |
| H618 HDMI and decode never reach mainline; vendor kernel is abandoned | High | v0 board becomes a dead end | Board spike in phase 2; v1 on CM5 or RK3588 where mainline already works |
| A major service blocks Linux browsers (Peacock already does) | Medium, ongoing | A "supported" tile goes dark | Weekly automated playback checks; tiles show ceilings; no service is promised in marketing copy |
| Netflix L3 ceiling drops from 720p to SD | Medium | Weakens the headline use case | Position YouTube, self-hosted media and casting as primary; subscriptions as secondary |
| CEC does not work on H618 | Medium | No TV-remote control on v0 | Validate week one; cec-gpio fallback; v1 boards have working CEC |
| DRAM prices keep rising | Medium | Built BOM exceeds $89 | Price target reviewed at gate 2; 1 GB tier as the floor model |
| Google Cast expectations | High | Reviews say "it is not a Chromecast" | Site says so up front; DIAL covers the YouTube case; Netflix dropped casting anyway |
| Miracast Wi-Fi Direct unsupported on the radio | Medium | Windows and Android mirroring missing | Ship as experimental; AirPlay and FCast cover most users |
| HomeKit badge without MFi | Low | Takedown request | No badge or trademark use; documented as community integration like Homebridge |
| Trademark collision with the existing OpenStick modem project | Low | Search confusion | Brand as OpenStick.tv everywhere; GitHub org openstick-tv |
| Solo-founder bandwidth across hardware, OS, site and 3D | High | Schedule | Phase gates are tests, not dates; the site's 3D hero is the one piece worth outsourcing |

**Open questions**

- [ ] Does any RK3588S board exist in a true stick form factor, or is the CM5 carrier the only stick-shaped v1 option?
- [ ] Flutter or web for the launcher at 1 GB RAM?
- [ ] Sell kits, bare boards with a flashed card, or files only?
- [ ] Which 3D artist or studio for the hero exploded-view model, if outsourced?

**Legal and licensing notes**

- Widevine CDM: distributed by Google inside official Chrome; OpenStick must not redistribute the CDM in its image. The image installs Chrome from Google's repo on first boot.
- Dolby Vision and Dolby Atmos decode are licensed; passthrough of bitstreams is fine, decode is not shipped.
- HDMI and HDCP: using the board's HDMI output is covered by the board vendor; OpenStick does not implement HDCP.
- HomeKit: HAP is open for non-commercial use; commercial use with Apple branding requires MFi. OpenStick uses no Apple marks.
- Google Cast: no receiver licence available; OpenStick does not emulate a Cast receiver for stock senders and does not use the Chromecast name.
- Open-source licences: OS image Apache-2.0, hardware files CERN-OHL-S, site and docs CC BY 4.0. UxPlay is GPL-3 and runs as a separate process.
- Naming: "OpenStick" is used by an unrelated modem-hacking project; no registered mark was found. Brand consistently as OpenStick.tv.
- Not legal advice; have a lawyer review the HomeKit and CDM points before kits ship.

## Sources

Pages opened for this PRD, grouped by section. Prices are as of 2026-10-20 and volatile.

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
- [Raspberry Pi memory-driven price rises](https://raspberrypi.com/news/more-memory-driven-price-rises)
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
