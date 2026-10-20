# OpenStick.tv PRD

Oct 20, 2026 · @Chris · Revision 4

> Revision 4: the device is **open hardware you can buy**. OpenStick designs and publishes the mainboard itself; a design house certifies that board, so units ship locked with full 4K, and owners can unlock their own unit (which drops subscriptions to 720p, like an unlocked Pixel). See [decision 0009](../decisions/0009-open-hardware-locked-by-default.md). The rest of revision 3 stands.
>
> Revision 3 changed direction: OpenStick must play Netflix, HBO Max, Disney+ and Prime in 4K. That is only possible on certified hardware with Widevine L1 and per-service certification, which requires a locked, vendor-provisioned platform. OpenStick therefore becomes a **certified Android TV device from a manufacturing partner, with an open layer on top**. The fully open Linux design (revision 2) is archived as [PRD.open-linux-edition.md](PRD.open-linux-edition.md); revision 1 as [PRD.original.md](PRD.original.md). Statements marked **(verify)** need confirmation from a partner, Google, or the services.

## Summary

OpenStick.tv is a 4K streaming stick you can buy, built on an open hardware design that is certified for Android TV, with an open layer that the big platforms don't offer: a launcher without ads, open casting receivers alongside Google Cast, Home Assistant, Matter and HomeKit control, a local API, and published enclosure and accessory designs. Subscriptions play at full quality because the device carries the certifications they require.

What changed and why:

1. **4K needs certification, and certification needs a locked device.** Widevine L1 requires a secure bootloader, a factory-provisioned device key and decryption inside the chip's secure area, under an agreement with Google. Netflix certifies devices through its own partner programme; HBO Max, Disney+ and Prime deliver 4K to certified Android TV / Google TV devices with L1 **(verify per service)**. An unlocked or user-modified device loses L1. So the core OS and DRM stack cannot be open or user-flashable.
2. **The fastest route is a manufacturing partner (ODM) that already holds the certifications.** ODMs building on Amlogic, Realtek or MediaTek TV chips ship certified Android TV / Google TV reference designs and manage Google, Widevine and Netflix certification for their customers **(verify with candidates)**. OpenStick brands the device and supplies the open layer.
3. **Google Cast becomes possible.** Certified Android TV devices can include the Google Cast receiver **(verify it is included in the partner's build)**. The open casting protocols (FCast, DLNA, AirPlay-compatible) are additions, not substitutes.
4. **Open hardware, locked software by default, owner's choice.** The mainboard, enclosure and accessories are OpenStick designs, published. Certification locks the software on the board, not the board design. Units ship locked so 4K works; an owner can unlock their own unit from settings and run any OS, losing L1 and therefore 4K subscriptions. Closed: Android core, DRM, Google services, streaming apps, and the device keys. The site says this plainly.
5. **The biggest open question is the Google tier.** Retail Google TV requires the Google TV home screen; Android TV Operator Tier allows a custom launcher but is negotiated with Google, normally for pay-TV operators **(verify)**. This decides whether the OpenStick launcher is the home screen or an app.

## Users and positioning

**Primary user:** someone who wants a top-tier 4K streaming stick but is tired of ad-filled home screens, wants their smart home to control the TV, and wants casting from any phone, not only Android and Chrome.

**Secondary:** self-hosters (Jellyfin, Plex, Home Assistant) who want one device for both subscriptions in 4K and their own media.

**Alternatives this is measured against**

| Alternative | Why someone picks it | What OpenStick must do better |
| --- | --- | --- |
| Google TV Streamer | Certified, 4K, Cast, Google Home hub | No ads on the home screen, open casting, Home Assistant and HomeKit control, local API |
| Apple TV 4K | Polish, privacy reputation, AirPlay | Price, Android app catalogue, Home Assistant, Google Cast |
| Fire TV, Roku | Price, 4K | No ads, no tracking-driven home screen, open integrations |
| Nvidia Shield | Best Android TV box | Current hardware, longer updates, smart-home integration |
| Projectivy-style launcher on a Google TV device | Free, removes ads | A device where the clean launcher is the default and integrations are built in |

## Goals and non-goals

**Goals (v1)**

- 4K HDR streaming of Netflix, HBO Max, Disney+, Prime Video, Apple TV+ and YouTube, with Dolby Vision and Atmos where the platform licence covers them.
- Google Cast receiver plus open receivers: FCast and DLNA in v1, AirPlay-compatible receiver if legal review allows.
- A launcher without ads or sponsored rows, as the home screen if the platform tier permits (see Platform tier).
- Home Assistant, Matter and HomeKit control of the TV; a documented local API.
- HDMI-CEC, wall power, Wi-Fi 6, Bluetooth remote with voice.
- OpenStick's own software collects no analytics. The device's Google and streaming components are disclosed clearly, with every privacy control the platform allows turned to the most private setting by default.
- Open hardware: published mainboard schematics and layout, enclosure, remote shell and accessory CAD; published source for every OpenStick app and service.
- Owner unlock: a settings switch (off by default, wipes the device, shows a clear warning) lets owners unlock the bootloader and flash any OS **(verify Google's Android TV rules for unlockable bootloaders)**.
- Security updates for at least five years, in the partner contract.

**Non-goals (v1)**

- An open stock OS. The shipped OS is locked; owners may unlock, accepting the loss of 4K subscriptions.
- Build-it-yourself kits for the 4K edition. Device keys are provisioned in the factory, so certified units are sold finished. (People can still build from the published files; their build runs at 720p.)
- Custom silicon.
- Replacing Google Play or Google Cast.

## Platform

**Chip**

A current Android TV chip with AV1, VP9, HEVC hardware decode, HDR10/HDR10+/HLG/Dolby Vision, HDMI 2.1-class output, Widevine L1 support and a certified reference design. **New hard requirement: the chip vendor must allow OpenStick to publish its board schematics and layout.** Amlogic, the usual Android TV vendor, keeps datasheets and reference designs under NDA; Rockchip publishes more but has fewer certified Android TV designs; Realtek and MediaTek are similar to Amlogic **(verify each vendor's terms)**.

**Platform tier: the decision that shapes the product**

| Tier | Home screen | Google services | Fit for OpenStick |
| --- | --- | --- | --- |
| Google TV (retail) | Google TV home, mandatory | Full, Google account at setup | 4K and Cast, but the ad-free launcher is only an app the user opens; weakest differentiation |
| Android TV Operator Tier | Custom launcher allowed | Full, with operator controls | Best fit: OpenStick launcher as the home screen; normally granted to pay-TV operators, so needs a deal with Google **(verify eligibility)** |
| AOSP Android TV without Google, with L1 and direct service deals (the Fire TV model) | Fully custom | None | Most open layer, but every streaming app needs its own distribution deal; not realistic for a new company |

Plan: ask each ODM which tiers it can certify for a new brand, and approach Google about Operator Tier in parallel. If only retail Google TV is available, OpenStick still ships, with the launcher as an app, and the rest of the open layer unchanged.

**What is open and what is not**

| Layer | Open? | Notes |
| --- | --- | --- |
| Mainboard design | Yes, CERN-OHL-S-2.0 | Schematics, layout, BOM; subject to the chip vendor's terms |
| Bootloader, kernel, Android framework, DRM | Locked on shipped units | Owner can unlock (loses 4K). GPL kernel source published |
| Device keys (Widevine, attestation) | Never | Provisioned in the factory inside the chip's secure area |
| Google services, Play Store, Google Cast | No | Google |
| Streaming apps | No | Each service |
| OpenStick launcher | Yes, Apache-2.0 | System app on the device; source on GitHub |
| OpenStick services (local API, integrations, casting receivers) | Yes, Apache-2.0 | System or privileged apps |
| Home Assistant integration | Yes | HACS, then HA core |
| Enclosure, remote shell, accessories | Yes, CERN-OHL-S-2.0 |  |

## Parity matrix

Benchmark: [Google TV Streamer (4K, 2024)](https://store.google.com/product/google_tv_streamer_specs).

| Capability | Google TV Streamer | OpenStick v1 | Verdict |
| --- | --- | --- | --- |
| 4K60 HDR output | Yes | Yes | Match |
| HDR10 / HDR10+ / Dolby Vision / HLG | All four | All four if the partner holds the Dolby licence **(verify)** | Match |
| Dolby Atmos | Yes | Yes, passthrough and decode per platform licence | Match |
| 4K Netflix, HBO Max, Disney+, Prime | Yes | Yes, once certified | Match |
| Google Cast receiver | Yes | Yes **(verify in partner build)** | Match |
| Open casting (FCast, DLNA, AirPlay-compatible) | No | Yes | Better |
| Home screen without ads | No | Yes on Operator Tier; as an app on retail Google TV | Better or partial |
| Voice remote and assistant | Google Assistant / Gemini | Google Assistant on the platform; remote with mic | Match |
| HDMI-CEC | Yes | Yes | Match |
| Ethernet | Built-in | Via USB adapter or a dock **(decide)** | Partial |
| Bluetooth, Wi-Fi | BT 5.1, Wi-Fi 5 | BT 5.x, Wi-Fi 6 | Match or better |
| Matter / Thread | Matter hub and Thread border router | Matter controller via Google Home on the platform; Thread only if the partner board has the radio **(decide)** | Partial |
| Home Assistant control | Via third parties | Built-in integration and local API | Better |
| HomeKit control | No | Planned (Television accessory) | Better |
| Guest mode | Removed in 2023 | Yes, for open casting receivers | Better |
| Five years of updates | Yes | In the partner contract | Match if contracted |
| USB | None | USB-C with OTG | Better |

## Streaming services

All major services should deliver 4K once the device is certified. Requirements per service **(verify each)**:

| Service | What it needs | Owner of the work |
| --- | --- | --- |
| Netflix | Netflix device certification, Widevine L1, HDCP 2.2 | ODM with Netflix; OpenStick may need to be a Netflix partner brand |
| YouTube | Google TV / Android TV certification; YouTube app from Play | ODM via Google |
| HBO Max, Disney+, Prime Video, Apple TV+ | Widevine L1 on a certified Android TV / Google TV device; apps from Play; some services run their own device approval | ODM, plus each service where required |
| Jellyfin, Plex, Kodi | Nothing; apps from Play | — |

Product rules:

- A service is advertised only after it plays at its top quality on a production unit.
- Weekly automated playback checks on production units (HDMI capture measures the real output) catch regressions after updates.

## Open layer

**Launcher.** A d-pad-first home screen written in Kotlin with Compose for TV. No ads, no sponsored rows, user-ordered apps, profiles, accessible (TalkBack, high contrast, text size). On Operator Tier it is the default home; otherwise it ships as an app.

**`openstickd` service.** An Android service app that owns the local API (REST plus WebSocket, mDNS `_openstick._tcp`), on-screen pairing, CEC actions through the platform's HDMI-CEC APIs, and the bridges below. Security rules unchanged from revision 2: pairing code, bearer token, `Host` check, no wildcard CORS, rate-limited pairing.

**Casting receivers.**
- Google Cast: built into the platform **(verify)**.
- FCast: the FCast receiver has an Android TV build; ship it preinstalled **(verify licence and maintenance)**.
- DLNA/UPnP renderer: open-source Android implementation, preinstalled.
- AirPlay-compatible receiver: no open Android implementation is production-ready, and Apple licenses AirPlay to TV makers through its partner programme. Decide between a licence through the partner and leaving it out **(legal review)**.
- DIAL: part of the platform's YouTube support.

**Smart home.**
- Home Assistant: HACS integration (`ha-openstick`) using the local API: media player, remote, power, notify-on-TV.
- Matter: Google TV devices can act as Matter controllers through Google Home. OpenStick additionally exposes the TV as a Matter video player if the platform allows it **(verify)**.
- HomeKit: Television accessory via HAP from `openstickd`. Commercial use needs MFi or a legal position (see Legal).

**Web app tiles (for unlocked and self-built units).** When an owner unlocks their unit, or builds a board from the published files, the native streaming apps lose 4K and may refuse to run. Web app tiles keep those services usable at the quality a browser is allowed:

- One tile per service (Netflix, HBO Max, Disney+, Prime Video, Apple TV+, YouTube), each opening the service's own website full-screen in Google Chrome with its own isolated browser profile. Google Chrome is required because its DRM module is the only one licensed for this; Chromium with a borrowed module is not allowed.
- Each tile is an installable web app (manifest and icon) so it launches like an app from the OpenStick launcher. A service worker only serves the tile's own launch shell and offline "no connection" page; it never touches the service's pages, video or DRM, and cannot raise quality.
- Remote control: the d-pad and media keys map to the keyboard and pointer input the sites already accept. OpenStick does not inject scripts into, restyle, or change the services' pages, and does not fake a TV or other device identity to them.
- Every tile shows its real ceiling ("Netflix, up to 720p in a browser"), using the same weekly automated checks as the native apps. HBO Max caps its website at 720p on every device; Disney+ offers no 4K in any computer browser; Prime Video gives SD in Chrome on Linux.
- On locked, certified units the native apps are the default and the tiles are hidden, except for a service with no Android TV app.
- Applies to: OpenStick's unlocked Android build and the open Linux build described in [PRD.open-linux-edition.md](PRD.open-linux-edition.md).

**Privacy.** OpenStick apps send nothing. At first run, a plain-language screen lists what Google and the streaming apps collect and links to each control; every optional Google setting (usage and diagnostics, personalised ads, ambient data) defaults to off where the platform permits **(verify which defaults the tier allows the brand to set)**.

## Hardware

- **Mainboard:** designed by OpenStick (or with a design house), derived from the chip's certified reference design, published CERN-OHL-S-2.0. Certified as the OpenStick model.
- **Form factor:** HDMI stick or puck with short HDMI cable, decided with the partner's existing designs.
- **Enclosure and remote shell:** designed by OpenStick, published CERN-OHL-S-2.0, manufactured by the partner. Must fit the partner's thermal and antenna constraints.
- **Remote:** BLE with microphone (Google Assistant), dedicated OpenStick home key, find-my-remote buzzer, IR for TV power and volume.
- **Power:** wall adapter included, certified (UL/CE).
- **Ethernet:** decide between a bundled USB-C Ethernet adapter, a dock, or none.

## Partner selection

OpenStick needs a **design house or ODM that certifies a customer-designed board** (not only its own white-label designs), plus a chip vendor whose terms allow publishing the board.

Criteria:

1. Has certified Android TV / Google TV devices for a recent chip, and will certify a board OpenStick designs and publishes.
1a. Accepts an owner-unlockable bootloader on the shipped device.
2. Handles Widevine L1 provisioning and Netflix certification, or has done so for other brands.
3. Will build a custom enclosure and remote and preinstall OpenStick system apps with the privileges they need (local API, CEC, launcher).
4. Commits in contract to security patches for five years, with OTA infrastructure and a process for OpenStick app updates.
5. Will consider Operator Tier.
6. Minimum order quantity and lead time that a new brand can meet **(record per candidate)**.
7. Publishes the GPL kernel source for the device.

Process: shortlist 3 to 5 ODMs, request certified reference designs and certification history, get engineering samples, evaluate against the criteria, sign.

## Success metrics

| Metric | Target |
| --- | --- |
| Certification | Netflix, YouTube, HBO Max, Disney+, Prime all at 4K on production units before launch |
| First-run | 90% of beta users reach the home screen without help |
| Update reliability | Zero bricked units from OTA |
| Integrations | Home Assistant and HomeKit control work for 95% of beta households |
| Privacy | Every OpenStick component audited as sending no analytics; disclosure screen shipped |

## Milestones

`Partner search → [gate 1] → Samples and platform tier → [gate 2] → Board design and open layer alpha → [gate 3] → Certification and beta → [gate 4] → Launch`

| Phase | Scope | Exit gate |
| --- | --- | --- |
| Partner search | Chip vendors that allow publishing schematics; design houses that certify customer boards; tiers offered, MOQ; approach Google about Operator Tier | A chip and a design house that both accept open schematics and owner unlock |
| Samples and platform tier | Engineering samples; 4K service tests on reference units; tier decision | Partner chosen; tier decided; contract signed |
| Board design and open layer alpha | OpenStick mainboard from the reference design, published; launcher, `openstickd`, local API, FCast and DLNA receivers, HA integration, HomeKit bridge, privacy disclosure; running on samples | All OpenStick apps pass partner and Google requirements for preinstalled apps |
| Certification and beta | Partner runs Google, Widevine, Netflix certification on the OpenStick SKU; enclosure and remote tooling; FCC/CE; 50 to 100 beta units | All certifications passed; beta blockers closed |
| Launch | Production run, store, support | Units shipped |

## Website

The site stays on Astro, Starlight and React Three Fiber. Changes:

- The exploded view shows the real device teardown, built from the published board and enclosure files.
- The "Build it" path becomes "Docs": setup, integrations, local API, and the open-source components.
- The service chart shows 4K for certified services, and states plainly which parts are open and which are not.
- "Buy" appears only once units exist.

## Risks

| Risk | Likelihood | Impact | Mitigation |
| --- | --- | --- | --- |
| No chip vendor allows publishing schematics | Medium-high | Board can't be fully open | Ask every vendor early; fall back to publishing everything except NDA-covered parts, stated plainly |
| No design house will certify a customer-designed, owner-unlockable board at viable volume | Medium | No product | Shortlist several; fall back to a white-label certified board with open case and apps |
| Certifying a new board design costs more and takes longer than a white-label | High | Budget and schedule | Base the board closely on the certified reference design |
| Operator Tier not available | High | Launcher only as an app; weaker differentiation | Lead with integrations, open casting and privacy defaults; launcher as an app still works |
| Netflix certification delay or refusal | Medium | Headline feature missing | Make it a launch gate; partner's certification history is a selection criterion |
| Google requirements conflict with privacy claims | High | Claims become misleading | Claims cover OpenStick software only; disclosure screen; legal review of all privacy copy |
| Preinstalled OpenStick apps denied the privileges they need (CEC, home screen, background service) | Medium | Integrations degrade | Agree privileges in the partner contract before signing |
| AirPlay receiver legality | High | No iPhone casting | Licence via partner or omit; FCast and Google Cast cover most phones |
| Partner stops updates early | Medium | Five-year promise broken | Contractual commitment with escrow or penalty |
| Capital: MOQ, certification and tooling costs | High | Cannot reach launch | Budget and funding plan before gate 2 (pre-orders or crowdfunding) |
| Solo founder bandwidth | High | Schedule | Partner does platform work; OpenStick focuses on the open layer and brand |

## Legal and licensing notes

- Widevine, Google TV / Android TV, Netflix: all through the partner's and OpenStick's agreements with Google and each service. Nothing extracted or redistributed outside those agreements.
- GPL: the partner publishes kernel and other GPL source for the device; OpenStick links to it.
- OpenStick software: Apache-2.0. Enclosure and accessories: CERN-OHL-S-2.0. Docs and site content: CC BY 4.0.
- HomeKit: commercial use needs MFi or a legal position; decide before launch.
- AirPlay: licence through Apple's programme via the partner, or omit.
- Matter: certification through CSA for any Matter device type OpenStick itself exposes.
- Trademarks: use service and platform names only as the respective programmes permit ("Google TV", "Chromecast built-in", "Works with Matter" each have rules).
- Privacy: privacy policy covering OpenStick apps, the update channel for OpenStick apps, and pointers to Google's and the services' policies.
- Product compliance: FCC, CE/UKCA, RoHS, WEEE, battery rules; usually handled with the partner, but OpenStick as the brand owner is responsible.
- Not legal advice.

## Open questions

- [ ] Which chip vendors allow publishing board schematics?
- [ ] Which design houses certify a customer-designed board, at what MOQ?
- [ ] Does Google's Android TV certification allow an owner-unlockable bootloader?
- [ ] Do the services' terms allow launching their websites as full-screen web app tiles on a TV device?
- [ ] Can OpenStick get Android TV Operator Tier?
- [ ] Netflix: does the partner's certification cover a new brand, or must OpenStick join Netflix's partner programme?
- [ ] Which Google privacy defaults can the brand set?
- [ ] Ethernet and Thread: include or not?
- [ ] AirPlay: licence or omit?
- [ ] Funding model for MOQ, tooling and certification.

## Sources

Carried from revisions 1 and 2 where still relevant; new claims in this revision are marked **(verify)** until confirmed with partners, Google or the services.

- [Widevine DRM partner process](https://www.widevine.com/solutions/widevine-drm)
- [Netflix on uncertified devices](https://help.netflix.com/en/node/124156)
- [Google TV Streamer specs](https://store.google.com/product/google_tv_streamer_specs)
- [Google Cast receiver authentication analysis](https://yingtongli.me/blog/2019/12/20/gcast-auth.html)
- [FCast](https://github.com/futo-org/fcast)
- [Android Authority on Netflix dropping casting](https://www.androidauthority.com/netflix-casting-chromecast-google-tv-streamer-3620784)
