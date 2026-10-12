# Research notes: iPhone 18 Pro Max as a desktop PC

Last updated: 2026-10-12. Sources are web searches; many are third-party and some are unverified (flagged). See also `Running Desktop OS on iPhone.md` (initial research document).

## Goal
Use an iPhone 18 Pro Max as a desktop: Windows or macOS, full mouse and keyboard, multiple monitors, over one USB-C cable to a dock.

## Hard constraints
- Windows and macOS cannot be installed natively. Locked bootloader, signed-code-only model.
- Stock iOS 26 and 27 only mirror to an external display. iOS 27 (public Sept 14, 2026) added only a minor external-display refinement (window position memory). No iPhone desktop mode shipped.
  - MacRumors iOS 27 changes: https://www.macrumors.com/2026/06/10/apple-lists-250-changes-ios-27-and-more/
  - https://externaldisplayiphone.com/blog/did-ios-26-add-desktop-mode (extended-desktop code in iOS 26.1 was closed again in 26.2)
- Apps can take over the external display with a `windowExternalDisplayNonInteractive` scene (no touch interaction; input comes from phone-connected keyboard/mouse). Interactive extended windows are iPad-only.
  - https://developer.apple.com/forums/thread/773838
- The phone drives ONE external display. Multi-monitor docks rely on DisplayLink (needs driver, not on iOS) or MST (not supported on iOS/Apple silicon macOS). Bluetooth monitors do not exist; AirPlay/Miracast would add at most one laggy screen. Not verified specifically for the 18 Pro Max.
- A Mac cannot act as a monitor for the iPhone over USB-C (no video input). The MacBook can only be setup host, or view the phone via QuickTime / iPhone Mirroring.
- No hypervisor on non-jailbroken iOS. UTM's HV build is TrollStore-only and needs an M1 iPad or newer. A research repo claims the hardware hypervisor is on in iOS 26 but gated by an Apple-only entitlement (unverified).
  - https://github.com/utmapp/UTM/releases
  - https://github.com/cloverfield11/ios26-hypervisor-research
- Jailbreak: Dopamine 3.0 (Aug 2026) covers iOS 26.0/26.0.1 on A12/A13 only. iOS 27 usbliter8 bootrom exploit is a lab demo on iPhone 11 Pro. Nothing for A19/A20-class phones.
  - https://www.macrumors.com/2026/08/07/ios-26-dopamine-jailbreak/
  - https://cybersecuritynews.com/booted-jailbroken-ios-27-iphone-11-pro/

## Route A: Local Windows 11 ARM in UTM with JIT
- UTM (sideloaded full build, not App Store UTM SE) runs QEMU with JIT (TCG). External display output supported since 4.0.x; guest can go to the monitor at native aspect ratio; raw pointer capture.
  - https://newreleases.io/project/github/utmapp/UTM/release/v4.0.7
- JIT on iOS 26 / TXM devices: needs StikDebug (on-device debugger, pairing file, loopback VPN like LocalDevVPN) with a per-app script (UTM has the "legacy" script). Support on iOS 27 reported for betas but NOT confirmed. StikDebug reportedly removed from the App Store Jan 2026; use GitHub/AltStore/SideStore builds.
  - https://github.com/StikDebug/StikDebug/
  - https://github.com/Provenance-Emu/iCube/issues/11
- Evidence of speed:
  - iPad Air M2 + JIT + tiny11: "decent". Windows would not boot without JIT. https://www.windowslatest.com/2025/04/22/dev-runs-windows-11-arm-on-ipad-air-m2-using-utm-with-jit-and-its-decent/ and https://www.heise.de/en/news/Windows-11-for-ARM-runs-perfectly-on-iPad-Air-M2-EU-and-JIT-make-it-possible-10364121.html
  - iPhone 15 Pro + UTM SE (no JIT): "terribly slow", about 20 min boot. https://www.tomshardware.com/software/windows/windows-11-demoed-on-the-iphone-15-pro-tiny11-and-utm-se-used-to-create-a-terribly-slow-pocket-pc
  - iPhone 13: extremely slow (2022). https://github.com/utmapp/UTM/discussions/4763
  - NO iPhone 16/17/18 benchmarks found.
- Boot tip (Windows ARM stalls in installer): QEMU properties `highmem=off`, `virtualization=on`; cortex-a72, 2 cores, force multicore. https://github.com/utmapp/UTM/discussions/3419
- Sideloading: free Apple ID = 7-day certs, 3 app limit. Developer Mode required. RAM limit for the VM likely about 4-6 GB (estimate).
- macOS cannot run locally this way. Only classic 68k Mac OS via Mini vMac.

## Route B: Cloud / remote PC from the phone (works today)
- Windows 365 on USB-C iPhone (April 2026 test): works; mouse only with a Microsoft Bluetooth mouse on firmware 2.2+, enterprise licensing needed. https://www.niallbrady.com/2026/04/26/using-a-windows-365-cloud-pc-via-a-usb-c-connected-iphone/
- iPhone 17 Pro user: keyboard and display work in Windows App via hub, mouse not on external display. https://discussions.apple.com/thread/256182871
- Microsoft: AssistiveTouch mice not supported on external monitors in Windows App. https://learn.microsoft.com/en-us/windows-app/input-keyboard-mouse-touch-pen
- Parsec / Moonlight / macOS rented Macs: not researched yet for iPhone external display.

## Route C: Native iOS "desktop" apps
- XeOS (https://externaldisplayiphone.com/) and External Display Browser (App Store) draw their own desktop on the external display. Claims unverified.
- Dynamic Stage (jailbreak multitasking tweak) is EOL, iOS 14-16 only. https://www.idownloadblog.com/2024/04/01/dynamic-stage-ios-16-update/
- OpenDisplay is the reverse direction (iPhone as Mac second screen). https://github.com/peetzweg/opendisplay
- Phone demo: iPhone 15 Pro on Studio Display with infiniteX2P demo app, laggy. https://www.phonearena.com/news/With-some-tinkering-and-a-monitor-iPhone-15-Pro-can-be-used-as-a-PC_id150925

## Route D: Wine on iOS (found via r/EmulationOniOS, 2026-10-12)
Project READMEs read on 2026-10-12; not yet tested by us.
- **Madeira** (https://github.com/willfaust/madeira, GPL-3.0, ~2.2k stars, active): runs x86-64 Windows PC games on jailed iOS (no jailbreak). Stack: FEX-Emu (x86 to ARM64 translation) + Wine 11.4 built for ARM64EC (Wine itself runs natively, only the app's code is translated) + DXMT (D3D9/10/11 to Metal) + own D3D12-on-Metal layer. Single iOS process. Has a Windows desktop session, Steam login/install, Steam Cloud, keyboard/mouse/trackpad passthrough as real input, Bluetooth controllers. Requires iOS 26+ (dev on recent Pro iPhones), JIT via StikDebug or built-in StikJIT; on iOS 27 the built-in helper can pair the iPhone itself with no computer. Free Apple ID works (7-day signing). Not on the App Store because it needs JIT.
- **Iridium** (https://github.com/intraducine/iridium, AGPL-3.0, ~77 stars): experimental Windows-game runtime for iPhone/iPad built on the Madeira runtime plus its own forks of FEX/Wine; native game library, touch/keyboard/mouse/controller. Steam path only partly validated; builds not yet device-validated (per its README). Unsigned IPA build script included.
- **Husk** (https://github.com/Leviidev/Husk, ~1.5k stars): runs Android apps on iPhone. Translation layer (native arm64 Android code with Android libs stubbed; fast) or full LineageOS under QEMU (slow, no hypervisor on iOS). Possible Android-desktop angle; not researched further.
- **Manic EMU** (https://github.com/Manic-EMU/ManicEMU, AGPL-3.0): retro game emulator; relevant only as an example app distributed via StikStore/SideStore with AirPlay output.
- Implication: Wine on ARM64 (Route D) avoids booting a full emulated Windows 11, so it may be far faster than UTM for running individual Windows apps, but it gives Windows apps, not a full Windows OS, and is aimed at games. Compatibility with productivity apps unknown. Multi-monitor still unsolved.
- **MeloNX** (https://starforgejo.dev/projects/MeloNX, GPL-3.0, Nintendo Switch emulator based on Ryujinx; last commit 2026-10-11; CI on canary failing at that time): not a desktop route, but a good reference for the JIT tooling on iOS 26+. Requires JIT (StikDebug recommended on iOS 26), "Increased Memory Limit" entitlement on all devices, "Extended Virtual Addressing" on low-RAM devices. Recommended sideload tool: PlumeImpactor, optionally with SideStore for on-device refresh. No external display or mouse mentioned.
- **DukeX** (https://github.com/MaftyManicEMU/DukeX, original Xbox emulator built on xemu, ~135 stars, last push 2026-10-05): not a desktop route. Reference for JIT and display: TCG with version-aware JIT, iOS 26+ uses StikDebug with the "Universal.js" script assigned to the app's bundle ID, optional automatic StikDebug handoff, MoltenVK + native Metal presenter, AirPlay and external-display handling. Distributed via StikStore/SideStore source JSON. Requires A14/M1 or newer.
- **AetherPS4** (https://github.com/Leviidev/AetherPS4, same author as Husk; PS4 emulator for iOS/iPadOS built on shadPS4, GPL-2.0, ~73 stars, last push 2026-09-13): not a desktop route. Technical reference only: uses a FEXCore fork ported to Apple ARM64 hosts (x86-64 to ARM64 JIT, same family as Madeira), an iOS-specific JIT allocator with dual-mapped RW/RX memory, MoltenVK renderer, and input code for controller/keyboard/mouse. Needs StikDebug-style external JIT because sideloaded apps cannot request MAP_JIT. See its PORTING.md and runtime/ folder.
- **Madeira input and display findings (docs/KEYBOARD_MOUSE.md, code search, 2026-10-12):** USB or Bluetooth keyboard and mouse reach Windows programs as real events through GCKeyboard/GCMouse (HID to Windows virtual keys, 5 buttons, wheel, visible program cursor). Input only goes to the program while it has focus. On iPhone, UIKit does not report pointer position, so mouse focus relies on AssistiveTouch taps, pointer lock, or a held button. External display: no mention in README or docs, and a code search found only `UIScreen.main` uses and no external-display scene code (search may be incomplete). Likely outcome with a monitor: iOS mirrors the phone screen. Untested.
- Discord: Madeira has a community server linked from its README.

## Hardware
- Owned: UGREEN 70336 = 4-port USB-A USB-C hub, NO video output, no PD pass-through. Good only for keyboard/mouse/storage.
- Need for display test: a hub or adapter with HDMI 2.0 4K 60Hz, USB-C PD pass-through, 2+ USB-A, single cable to the phone. Check iPhone compatibility and return policy; none of the sources tested these with an iPhone.
- Candidate: Belkin Connect 5-Port USB-C Hub with 4K HDMI (AVC021). https://www.belkin.com/ca/p/5-port-usb-c-hub-with-4k-hdmi/AVC021fqBK.html (AU availability unchecked)
- Fallback: UGREEN Revodok 6-in-1 (PB Tech NBDUGR15852; avoid older 5-in-1, only 4K 30Hz). https://www.pbtech.com/product/NBDUGR15852/UGREEN-Revodok-6-in-1-USB-C-Hub-2x-HDMI---2x-USB-A
- Cheapest test: USB-C to HDMI 4K60 adapter plus any HDMI monitor/TV.
- Without a monitor: HDMI capture dongle into the Mac.

## Plan
- Phase 1a: try Madeira (Route D) first: sideload, JIT, open its Windows desktop session, test a Windows app and keyboard/mouse. Compare with Phase 1b.
- Phase 1b (Mac + phone, no monitor): sideload UTM, enable JIT via StikDebug, install Windows 11 ARM, measure speed. View via QuickTime or iPhone Mirroring. This tests the biggest unknown.
- Phase 2: dock/adapter + monitor: verify external display path, keyboard, mouse.
- Fallback: Route B (cloud PC).

## Open questions
- Test phone: iPhone19,3 (18 Pro Max) on iOS 27.0, paired with the Mac over USB-C (checked 2026-10-12). Developer Mode ENABLED and developer disk image services available (devicectl, 2026-10-12). Open: does StikDebug JIT work on iOS 27.0 for Madeira and UTM?
- Real Windows 11 ARM speed on an 18 Pro Max.
- Whether iPhone Mirroring on the Mac can drive UTM input.
- Multi-monitor: unsolved on iPhone; workaround is one ultrawide plus window zones.
- Parsec/Moonlight behavior on iPhone external display.
- Reddit threads not found by search; try searching r/UTMapp, r/EmulationOniOS, r/iosjailbreak directly.

## Tooling notes
- The Xcode iOS Simulator cannot test JIT, USB-C display output or sideloading; it is only useful for building/inspecting apps. Real-device tests need the physical phone.
- `xcrun devicectl` and libimobiledevice (`ideviceinfo`) both work with the phone over USB-C from the Mac.
