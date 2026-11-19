# Changelog

All notable changes to Nitrous are documented here. The GitHub release for a version is
published from the matching section of this file.

## 1.14.2 - @pra4u's fork

**@pra4u's fork.** Builds name themselves `Nitrous 1.14.2` above a `@pra4u's fork · Pure hardware control
for Acer Nitro Laptops.` line on the dashboard header card, and carry the same name in the window title
and the tray tooltip.

### Fixes

- **The CPU temperature is real now.** On this ANV15-52 the EC's CPU sensor (AcerGamingFunction
  `0x0101`) answers a fixed 25 C whatever the load - the GPU sensor and both fan tachometers on the
  same interface are correct. The CPU tile, the graph, CoolBoost and the fan curve now read
  `IA32_PACKAGE_THERM_STATUS` (MSR 0x1B1) through the PawnIO IntelMSR module Nitrous already ships,
  and keep the EC value on machines where that sensor is not available.
- **Opening the dashboard no longer spins the fans up.** With CoolBoost Auto on, the window asked
  for fan mode Auto as it was built - even though the tray already held it there - which put the
  EC's own fan curve in charge for the couple of seconds before the tray wrote the boost speed
  back. It now only changes the mode when the saved mode is something else.

## 0.14.1 - 2026-11-16

**pr4u_'s fork.** The build now names itself: `Nitrous 0.14.1` with `pr4u_'s fork` in the dashboard
subtitle, the window title and the tray tooltip, so a fork build is never mistaken for upstream.

### Dashboard

- **Squared, lightly curved cards.** Every container sits at a 4 px radius - cards, tiles, buttons,
  the power and fan switchers, dropdowns - instead of the old mix of pill and hard-corner shapes.
- **The CPU and GPU tiles are slimmer.** Each carries a 2 px accent edge across its top and shows
  just the label, load, fan RPM and a larger temperature; the shared two-minute temperature graph
  sits underneath them and plots both series on its own baselines.
- **The header rule is a scroll gauge.** It starts as a short accent line and runs out to the full
  width as you reach the bottom of a page. Its height never changes, so nothing shifts while you
  scroll.
- **No grey scrollbars.** Every page still scrolls by wheel, trackpad and touch, but the rail is
  hidden and the accent rule does the telling.
- **The active tab is marked three ways** - accent icon and label, a wash behind the tab, and an
  accent underline - while the inactive tabs stay legible against the dark rail.

### OPTIMIZATIONS

- **The card no longer asks you to guess.** It is titled `MEMORY & TEMP FILES` and splits into two
  labelled actions: `Free the ticked memory lists` with a **FREE MEMORY** button, and
  `Delete temporary files` with the measured size beside a **DELETE** button. The auto-run switch
  reads `Free memory automatically`, and its tooltip says temporary files are never removed on its
  own.
- Status lines say which tool ran and what it left alone: freeing memory reports
  "Freed 512 MB of memory. No files were deleted."

### Performance check

- **A bounded load that says what is limiting the machine.** `RUN CHECK` under POWER & PERFORMANCE
  loads every CPU thread for 60 s and reports the effective clock it settles at, the peak it started
  from, the drop between them, the package temperature and the fan speed - then names the limiter:
  a clock that falls on a cool chip is a power limit, one that falls at 92 °C or more is heat, and
  one that holds is neither. The dGPU is included when it is already awake and explicitly reported as
  asleep when it is not, because a measurement is never a reason to wake it.
- **It costs nothing until pressed.** No timer, no polling and no thread exists outside a run; the
  load threads stop on the button, on window close, or at the end of the minute, whichever is first.
- **`Nitrous.exe --perfcheck [seconds]`** runs the same check headless and writes the per-second
  readings to the log, so a run can be reviewed later or compared against a change.

### Fans

- **CoolBoost for Auto mode.** A switch under FANS that asks the firmware for extra fan on top of
  Auto: 6% once the CPU or GPU passes 52 °C, rising to 20% by 80 °C, tapered back when Auto is
  already loud, and handed back to the firmware below 45 °C. The boost is measured from the higher
  of the two curve targets, so a hot GPU under a cool CPU still gets air, and it ramps like the
  custom curve so it does not step audibly. Turning it on moves the fans to Auto and holds them
  there - Max and Custom are shut, and the fan hotkey says so - because CoolBoost has nothing to
  add to either. The mode the fans were on is remembered and comes back when the switch is turned
  off, so the Auto hold never quietly replaces a saved Max or Custom. It rides on the existing two-second fan engine: one temperature read per tick,
  cached curves, a write only when the speed really moves, nothing at all while the machine is
  cool, and a release on shutdown. Unlike the firmware CoolBoost card, this one works on models
  whose APGeAction function 7 is not CoolBoost, because it only asks Auto for more air rather than
  changing a firmware feature.
- **Curve defaults sit where the sensors actually are.** The CPU curves now start at 40 °C and the
  GPU curves at 45 °C instead of 30 °C, and the idle band asks for 10 to 20 points less at every
  profile. This generation idles the GPU at 52-54 °C and the CPU in the fifties, so a curve anchored
  at 30 °C was already asking for half its speed on a desktop. The top of each curve is unchanged,
  so nothing is given up under real load; Turbo no longer starts at 50 % before the first degree of
  heat.
- **Rises arrive in steps, not lurches - while the chip is cool.** A rise is immediate in intent,
  but below 55 °C it is handed over in at most 20 points per two-second tick, the allowance widens
  with heat (32 points at 60 °C, 43 at 65 °C), and from 70 °C it is not capped at all, leaving
  roughly 25 °C of headroom before throttle. A curve corner or a profile switch used to ask for
  thirty or forty points in a single write - that write is the step people hear. The widening
  matters more than the cap: a typical 20-40 point jump is one or two writes anywhere, and a
  full-scale jump is a single write across the entire warm band.
- **Small moves never reach the firmware.** A fan command is only written when a fan moves two
  points or more, so a curve walking down a degree at a time no longer sends a WMI write per tick.
- **Curves are parsed once, not every tick.** Both curves are cached per profile and rebuilt only
  when the saved value changes, and the interpolation is a plain scan over the already-sorted list
  instead of an `OrderBy().ToList()` per call. A curve tick is two cached registry reads plus two
  scans; CoolBoost reuses the same cache instead of keeping its own.
- **The curve editor costs less to drag.** The waypoint rings and dots are built once per point set
  and moved afterwards, instead of being destroyed and rebuilt on every mouse move; a drag also
  survives the pointer leaving the plot (the node at an edge used to be dropped), the value tooltip
  stays inside the graph, and SAVE & APPLY now enables the curve override it applies rather than
  leaving the manual sliders in charge.

### Updates

- **The fork no longer takes upstream's releases.** The updater was still pointed at
  `jeremyaliparo/nitrous`, so "Check for updates" would have offered - and installed - a stock
  Nitrous over this build, replacing the dashboard, CoolBoost, its Auto lock and the retuned fan
  curves with no sign that it happened. Checks now use the fork's own repository
  (`CHARVICK2/nitrous`), and when none is configured the check fetches nothing at all and says why
  instead of falling back to upstream. `UpdateRepo` under `HKCU\Software\Nitrous` overrides the
  slug without a rebuild; the resolved repository is listed in the diagnostics export.
- **Release tags must be versions to be offered.** A tag such as `1.14.1_Fork` is not comparable -
  read loosely it would look newer than the installed `0.14.1` and the updater would offer a
  downgrade to a build the installation has already superseded. Such tags are ignored with a note
  in the log rather than guessed at; releases tagged `X.Y.Z` (or `X.Y.Z-test.N`) are offered with
  their published SHA-256 checked before anything is replaced.

### Resource use

- **Registry reads are cached for 400 ms.** The dashboard was opening HKCU\Software\Nitrous about
  six times a second and the tray four times per tick; writes still invalidate the entry, so hotkeys
  land as before.
- **Profile sync runs at 1 Hz instead of 2 Hz**, halving the display and refresh-rate probes behind it.
- **The memory line repaints only when the number changes**, instead of rebuilding a string and
  re-laying-out text twice a second.
- **Battery WMI reads every 5 s instead of 2 s** - charge and discharge rates do not move faster than
  that, and each read constructed a ManagementObjectSearcher.
- **Cold pages are handed back**: the tray trims its working set after two minutes and every 30, and
  the dashboard trims whenever it is hidden. Measured here: tray 64 MB -> 30 MB after a trim,
  dashboard CPU 6.9% -> 5.0% of one core while open.

### Fixed

- **A duplicated temperature graph.** The dashboard briefly carried sparklines inside both tiles and
  the shared chart below. The tiles show numbers; the chart shows the history.

## 0.14.0-test.3 - 2026-11-14

### GPU

- **The NVIDIA hotkey registers on every machine now.** Its registration had been placed inside the
  lighting branch, which only runs on four-zone keyboards, so a single-colour keyboard (lori's
  AN517-51) never bound it - which is why the button worked while the hotkey did nothing: the
  dashboard finds the Control Panel on its own.
- **A failed detection is no longer remembered for the session.** The tray starts at logon, when the
  Start apps list can still be empty; caching that miss kept the hotkey and the tray entry away until
  the next restart. Only a hit is cached now, and a press retries the search.
- Every hotkey logs `Hotkey: <name> bound to <binding>.` when it registers, so a binding that does
  nothing is visible in the log instead of silent.

## 0.14.0-test.2 - 2026-11-14

### GPU

- **The NVIDIA Control Panel is found on machines that only have the Store build.** lori's laptop has
  no NVIDIA App, and its Control Panel is the packaged version: no exe under Program Files and no
  Start Menu shortcut, so the GPU card never appeared. Detection now also searches the vendor folder
  for `nvcplui.exe` by name and, as a last resort, asks the Start apps list - the only place a
  packaged app appears - then launches it through `shell:AppsFolder`. The log records what was found
  and which route found it.

## 0.14.0-test.1 - 2026-11-14

### GPU

- **Open the NVIDIA App from Nitrous.** A card at the bottom of the GPU page opens the NVIDIA App,
  or the older NVIDIA Control Panel on machines that still have it, with a rebindable hotkey
  (**Ctrl + Alt + N** by default). The card, the tray entry and the hotkey only exist when the
  software is actually found: the App's install path first, then the Start Menu shortcuts, which also
  cover the packaged Control Panel. Nitrous launches it and stays out of the way - driver settings
  remain the vendor's business.

## 0.13.1 - 2026-11-13

### Update

- **An update no longer leaves Nitrous stuck in Task Manager.** The updater replaces the running
  executable by renaming it, and a single-file app reads its assemblies from that file, so anything
  the old process had not loaded yet becomes unloadable. With GPU clock management off (the default)
  the first such load was the GPU manager during shutdown: teardown threw, the final `Exit` was
  skipped, and the process stayed alive holding the single-instance lock. The replacement then waited
  a minute and gave up, leaving a tray-less `Nitrous.exe` and a relaunch that refused to start until
  it was killed by hand.
- Shutdown now exits whatever happens, every disposal is guarded, and a restart that cannot spawn its
  replacement stays open instead of exiting into nothing.
- A replacement that has waited 30 s for the instance it is replacing now ends that instance and
  takes over. **Updating from 0.13.0 therefore pauses for about 30 seconds after the download**, with
  no tray icon, while that happens; it is the handover, not a freeze. From 0.13.1 onwards the update
  is immediate again.

## 0.13.0 - 2026-11-13

### Appearance

- **Background presets.** SETUP > APPEARANCE gains a Background list: **Midnight** (the shipped dark
  grey), **Black**, **Graphite**, **Slate** and **Navy**. Each preset rebuilds the card, hover,
  border, grid and toggle shades around the window colour, so text contrast holds and nothing loses
  its edge. Presets rather than a colour picker because those surfaces are built to match the window
  colour, and an arbitrary pick could leave the labels unreadable.
- **Sections.** A second list sets the card surface on its own: **Default** keeps the card that
  belongs to the chosen background, or pick Black, Graphite or Slate. The shades that sit on the card
  are lifted from it, so a section colour never leaves its controls without an edge. Background and
  Sections are stored separately and both are restored at startup.
- **Font.** A third list sets the UI font, applied to the dashboard and the fan-curve window alike.
  It is built from the fonts this machine actually has: Windows default, Segoe UI, Bahnschrift,
  Calibri, Tahoma, Trebuchet MS and Inter, plus Cascadia Code, Cascadia Mono, Consolas, JetBrains
  Mono, Iosevka NF and Fira Code as code faces when they are installed. A family that is not
  installed is not offered, and a saved choice that has since been removed falls back to the Windows
  font and says so in the log.
- The header subtitle now wraps instead of being cut at the card edge, and the fan card's caption row
  has its own column, so a wide or monospaced font no longer clips either.

### Cooling

- **CoolBoost, where the firmware has it.** A switch on the DASH fan card raises Acer's own fan
  speeds. It appears only when APGeAction function 7's marker says that function is CoolBoost (the
  AN517-51 answers 1, the 16S and the ANV15 line answer 0). On firmware that uses that function for
  the operating mode the control stays hidden, because writing it there would change the mode.
- The switch hides in Max, which already runs the fans at full speed, and the state is read back
  from the firmware after every write and every profile change, so it cannot show a state the
  firmware refused.

### Docs

- The README claimed compatibility with 2021+ models; a 2019 **AN517-51** runs it, added to the
  confirmed list. The compatibility text now says the protocol decides, not the year.


## 0.12.0 - 2026-11-13

### Dashboard

- **CPU and GPU usage** now sit beside the fan speed on the two cards: CPU from the idle/total time
  between polls, GPU from the same NVML reading the GPU page uses. GPU usage reads `--` while the
  dGPU is in D3, because Nitrous will not wake it to take a measurement.
- **A two-minute history** under the cards: 60 samples at one per 2 s poll, 20-100 C, right-aligned
  so the newest sample is at the edge. A sleeping dGPU leaves a gap rather than a drop to zero.
- The history also reports the **peak** in its window - highest CPU and GPU temperature, highest fan
  speed and highest usage - reset as the window rolls.

### Setup

- **Custom boot logo.** SETUP > STARTUP shows the picture Acer's firmware reads at power-on and
  replaces it with your own. Any picture is accepted and re-encoded to a baseline JPEG (or a GIF if
  the encoder produces anything else), scaled to at most 40 % of the panel so the firmware's decoder
  takes it. REMOVE deletes the picture and brings Acer's logo back. The card only appears where the
  firmware advertises the feature: SMBIOS record 0x0D or the BIOS setup's switch answering 0/1.
- The write is a file on the EFI system partition, not a BIOS change: temp file, flush, rename,
  read-back hash check and a free-space guard, and the firmware switch is remembered so REMOVE puts
  it back. Wiping or reinstalling the OS removes the picture with the partition.
- **Battery health**: design capacity, what the pack holds now and the cycle count, in the same card
  as the 80 % charge limit. Read once when the dashboard opens, from the battery's WMI classes, with
  `powercfg /batteryreport /xml` as the fallback for drivers whose `BatteryStaticData` class fails to
  instantiate (the AN16S-61 is one).
- The page reads as sections now: one spacing rule for the labels, one gap between cards, and the
  battery rows are a single block. `EXPORT DIAGNOSTICS` became `EXPORT ZIP`, which is what it does
  and no longer clips its own label.

### Lighting

- **The saved lighting is re-applied after a power-mode change**, two seconds later so Acer's
  acknowledgement has finished. The firmware lights the keyboard on a mode change even when the
  saved state is off, and on models with a controller the flash left its colour behind; until now
  nothing put the preset back until the next manual edit. The same restore runs at startup.
- **The Fn-brightness hint no longer returns after a slider is moved.** The sliders write the
  lighting controller while the Fn keys write a separate EC register, so moving a slider left the
  register untouched and the hint reappeared on the next open. A slider move now applies its level
  to that register too - snapped to its five steps and debounced, because each write opens the EC
  HID - and both sides are compared on those same five steps.

### Diagnostics

- **EC probe version 13** asks the firmware about CoolBoost the way Acer's own software does:
  APGeAction function 7's marker (1 = CoolBoost, 6 = the operating mode) and, when it is CoolBoost,
  sub-function 2's state. Both answers and a verdict go into the JSON and the text report.
- The boot-logo preview no longer throws when the EFI partition cannot be read, which happens on a
  dashboard opened without elevation; it shows no preview and logs the reason instead.


## 0.11.0 - 2026-11-09
<!-- release-name: Total Lightning Control -->

### Lighting

- **Screen sync.** The four zones follow the colours on screen, with a **Screen saturation** slider
  (0-200 %), a brightness cap and a **Follow** target (`Keyboard` or `Keyboard + lid logo`). It is a
  software effect: the tray keeps writing while the dashboard is closed, and turning it off restores
  the saved preset. Protected content and true fullscreen can block the capture; it is not a
  firmware effect.
- **Pulse with sound** on top of screen sync: the screen supplies the colours while the system audio
  scales their brightness, with a **Silence floor** (0-10 %) so hits can fall fully dark or keep a
  dim glow.
- **Sound sync** is its own source, taken from WASAPI loopback (system audio, never the microphone).
  **Peak** follows loudness, **Spectrum** splits bass / low-mid / high-mid / treble across the
  zones, **Beat** lands on kick onsets, and **Vibe** is a three-layer engine ported from the
  MIT-licensed KB-Breathe. A shared **Sensitivity** slider sets the threshold, and 16-, 24- and
  32-bit mix formats are accepted.
- **Vibe is the default on five-step keyboards.** A kick holds step 4 for about 110 ms and then
  drops straight to the base level, so the backlight goes 4 -> 0 instead of walking 4-3-2-1. Hi-hats
  give 3, sustained vocals 1-2, and silence gives 0. Levels map 1:1 onto the firmware's 25 % steps.
- **The active preset is the sync base.** Global contributes its colour and effect, Area its zone
  colours and Dynamic effect, and brightness follows that preset's slider. Switching Global/Area no
  longer suspends sync; the source is remembered and resumed.
- With sound sync on, the **firmware keeps animating the selected effect** while the audio drives
  its brightness, and both effect and speed stay editable. The **lid is written once as Static**,
  because the cover logo stutters when its brightness or speed is rewritten per frame.
- Sync **pauses for 1.5 seconds during a power-mode flash** so the controller's acknowledgement is
  not overwritten, and it keeps working from Area / Keyboard as well as Global.
- The **lid-logo check writes green** and asks again once for older answers: red is already in the
  power-mode palette, so it proved nothing on a Performance machine. A lid that only follows the
  power-mode key is explained and its controls are hidden.
- **Fn brightness is respected.** The LIGHT page reads the EC level once on open and, if it differs
  from the saved preset, says so under the slider instead of overwriting it. The saved preset is
  applied once when the tray starts, not every time the dashboard opens.

### Keyboard

- **Backlight brightness on models without a lighting controller**, for example the single-colour
  ANV15-52: a five-step row (`Off`, `25`, `50`, `75`, `100`) in Setup > KEYBOARD, using the EC HID
  interface where the model has one and `APGeAction` where it does not. Values snap to the
  firmware's levels, and toggling auto-off no longer resets the brightness.
- **Sound sync on that same five-step path**: Setup > KEYBOARD > **Pulse with sound** drives the
  backlight from the system audio with **Vibe**, **Beat** or **Peak**, and restores the chosen level
  when it stops. The pulse reaches 0 between hits rather than stopping at 25 %.

### Hotkeys

- Four new global bindings: **Cycle Fan Mode** (Auto -> Max -> Custom), **Screen Sync**,
  **Sound Sync** and **Next Lighting Effect**. All are unbound by default, confirm with an OSD, and
  the two lighting ones are hidden where the keyboard has no RGB zones.
- The **Global Hotkeys** card is grouped by where each action applies (WINDOW, DASHBOARD, LIGHT
  PAGE), with a separator and a one-line description for every binding.

### Dashboard

- The **Whisper Mode** label follows the request immediately, so it no longer vanishes while Quiet
  is still asking for it.

### Website

- An **interactive simulator** renders the dashboard, the four zones, the lid logo and the sync
  behaviour in the browser, linked from the header and the hero.
- The README and site describe the screen and audio sync as **TOTAL LIGHTNING CONTROL**.

### Diagnostics

- **EC probe version 12** records the keyboard type and both brightness paths, plus which one the
  model uses, so coverage can be mapped from a report without writing anything to the hardware.

### Build

- The executable carries **Jeremy Aliparo** as company and author, a copyright line and a one-line
  description.

## 0.10.0 - 2026-11-08

### Lighting

- The `KB` page is now `LIGHT`, and it drives the lid's cover logo as well as the keyboard. Both use
  the keyboard's own HID controller: no Acer service, driver or pipe is involved.
- **Global** and **Area** are two presets. Opening one applies it: Global writes both devices, Area
  re-applies the keyboard and the logo from their own settings. They keep separate settings, and the
  preset in use is remembered and applied on the next launch.
- **Global speed has five steps, each a matched pair** captured from the OEM software: the keyboard
  takes 1/3/5/7/9 while the cover logo takes 1/2/3/4/5. Sending the same number to both made the two
  drift apart above 5, because the logo has five steps and the keyboard nine.
- Colour applies to **Static** only. The animated effects run their own palette, so Global greys its
  colour to `Auto` and the Dynamic panel does not offer one.
- **Static** keeps the four per-zone colours; **Dynamic** sets one effect across all four with its
  own speed (1-9). Switching between them applies the selected one immediately, and Dynamic no
  longer lists Static, which is not an animation.
- The Area tab is a single card: the Keyboard/Cover logo selector sits at the top under a `DEVICE`
  label and a hairline, then the mode selector, then the settings. Brightness comes before Speed in
  every view, and the segmented buttons carry icons.
- A **single-colour keyboard** gets one Colour chip instead of four zone swatches that cannot work.
- The cover logo supports `Static`, `Breathing` and `Neon`; Breathing and Neon cycle their own
  palette (the colour reads `Auto`) and the logo has five speed steps. Selecting an effect it does
  not implement leaves it on its previous mode, which is what the OEM software does.
- **The lid logo is confirmed once.** On the first visit to Lighting, Nitrous writes a test colour to
  the lid and asks whether the "N" lit up. The answer is remembered, nothing further is written if it
  did not, and the check can be repeated with **TEST LID LOGO** or **CHECK AGAIN**.
- **Power mode change flash**, on by default, reproduces the OEM acknowledgement: the keyboard blinks
  in the mode's colour, and the firmware brings the power/mode key and the lid with it. The
  controller looks the colour up in a fixed palette, so the values Acer sends are used - white,
  orange, red and violet; anything else falls back to orange.
- The keyboard and the cover logo animate on independent controllers, so their pulses are not
  phase-locked. The Global card says so rather than promising a synchronisation the hardware does
  not have.
- Keyboard backlight auto-off moved to Setup, next to the hotkeys.
- A model with no reachable keyboard-lighting controller gets **no LIGHT tab** instead of a page of
  disabled controls. Its Fn keys may still change the backlight, but no user-mode interface exposes
  that, so there is nothing a control could do.

### Appearance

- Nitrous takes its accent colour from seven fixed accents on the Setup page, plus `Auto` (follow
  Windows) and a **custom picker for any colour**. The choice is remembered; a custom colour is
  stored as its hex.
- Custom colours added with **Add to Custom Colors** are remembered across sessions and shared by
  every colour picker: accent, Global, keyboard zones and cover logo.
- The accent is contrast-checked before use, and every colour in the interface comes from one
  palette rather than ~270 literals. Status colours stay fixed, so a temperature crossing a
  threshold looks the same whatever accent you pick.
- The GPU page's **APPLY** button follows a live accent change instead of keeping the one it started
  with.

### Hotkeys

- `Ctrl + Home` opens the dashboard out of the box, seeded only on a machine that has never stored a
  dashboard binding, so a binding you cleared stays cleared.
- Cycling the power profile stays unbound by default: changing a machine's power mode from a key
  press you forgot you set is a worse surprise than having no binding.
- The binding boxes live on the Setup page and are taller, so a shortcut reads as a key rather than
  as a text field that happens to contain one.

### Quiet mode

- Quiet now also tells NVIDIA's driver that a client asked for WhisperMode, which is what the NVIDIA
  App reads before it offers its WhisperMode controls. It does not change the power cap or the fans.
- The POWER row says `Whisper Mode: On` while it is requested, and the label disappears as soon as
  another mode releases it.

### Setup

- A new `KEYBOARD` section holds the global hotkeys, Sticky Keys and the keyboard backlight auto-off.

### Updates

- A test build is told plainly what it is when a release exists: "You're running a test version of
  Nitrous (…). The release version … is available on GitHub." Stable installs keep the prompt they
  have always had, and a release outranks a test build of the same version.

### Diagnostics

- EC probe version 11 lists Acer's own HID interfaces (`1025`, `0CF2`) and the keyboard controller
  vendors Acer ships (`04F2`, `05AF`, `0D62`). That separates "this model has no lighting
  controller" from "this model's keyboard is a USB device Nitrous does not speak".

### Not in this release

- **PL1/PL2 are not adjustable.** They need MSR `0x610`, which is ring0, and shipping a kernel driver
  for one setting would change what Nitrous is. The EC table some models expose is a capability
  record, not the applied limits. ThrottleStop remains the right tool for that.

## 0.9.0 - 2026-11-01

Nitrous 0.9.0 is the first release for external testing. It is not a stable release: several
controls depend on firmware that varies between models, and a few are still out of reach.

### Power modes

- `QUIET / BALANCED / PERFORMANCE / TURBO` now actually change the laptop's operating mode, and
  the enforced GPU power limit follows it. On the AN16S-61 that is 80 / 85 / 100 / 115 W.
- Nitrous used to write the Acer WMI misc register for this. The firmware stores that value and
  echoes it back but does not apply it, so the buttons looked like they worked and changed
  nothing. On models with an embedded-controller HID interface the mode is set there instead, and
  the mode is read back after every change so a refused one is never shown as active.
- Models without that interface keep the WMI path.
- That path is not a fallback in name only. On an ANV15-41, which has no embedded-controller
  interface, `Performance` moves the enforced limit to 65 W ΓÇö confirmed with `nvidia-smi`, not
  assumed from the register.
- Turbo is offered whenever the channel Nitrous actually uses lists it, not only when the WMI
  register advertises it, so models that used to hide the button now show it. A failed read is
  retried instead of being remembered for the session.

### Fans

- Fan curves rise instantly but no longer fall instantly. When the target drops, the fans hold for
  3 seconds and then step down at a chosen rate, so a short burst of load no longer makes them rev
  up and drop again.
- **Ramp down** in the curve editor: `Off`, `Gentle` (5 %/s), `Balanced` (10 %/s, default) and
  `Fast` (20 %/s). `Off` keeps the previous behaviour.
- The ramp never applies at or above 90 ┬░C: the fans go straight to the curve's speed.
- The curve editor now uses the dashboard's colours instead of the old blue theme.

### Keyboard lighting

- The `KB` page can set the keyboard's four lighting zones, an effect, and brightness. Static
  colours are per zone; an effect applies to all four.
- An `ALL` tile beside the four zones sets every zone to one colour in a single pick, which is
  what most people want. The tile previews the four zones as a gradient, so a mixed setup is
  visible without opening each one.
- This talks to the keyboard's own HID controller (an ENEK interface). The Acer WMI lighting
  methods store values the firmware never applies, so they were not usable for this.
- On models without that controller the card appears disabled, so an absent feature is not
  mistaken for a broken one.
- The four zones are also withheld when SMBIOS reports a single-colour keyboard, so a model that
  can only light one colour is not offered four swatches that cannot do anything.
- **Auto-off** turns the keyboard backlight off after a chosen time (`Never`, 30 seconds, 1 minute,
  5 minutes). The controller takes brightness and auto-off in one write, so Nitrous reads the
  current level first and keeps it rather than guessing.
- The auto-off is now read back after it is written. A controller that answers "done" but has
  nowhere to store the value is disabled with an explanation, instead of accepting the change and
  doing nothing.
- Opening the dashboard re-applied the lighting, which switched the backlight back on without
  re-arming the auto-off, so the keyboard stayed lit until the timeout was picked again. Any
  lighting change now re-arms the saved auto-off, debounced so dragging the brightness slider
  costs one write rather than one per tick.
- Where there is no embedded-controller interface, the same 30-second auto-off is a firmware timeout
  reachable over Acer's `APGeAction`. Nitrous sets that too and reads it back, so the feature now
  works on models that only expose it that way. That path is on/off at 30 seconds rather than a list,
  and the backlight returns on the next key press after the timer has already switched it off ΓÇö
  the firmware's behaviour, not a Nitrous choice.

### Accessibility

- **Sticky Keys** on the `KB` page: the Windows setting that lets you press one key at a time for
  shortcuts. It is a per-user preference rather than firmware, so it works on every model. The flags
  are read from Windows before changing them, so the confirmation and sound preferences are kept.

### Battery and USB

- **USB Charging While Off** on the Setup page: keeps charging USB devices while the laptop is
  off, stopping at 10 %, 20 % or 30 % battery. Nitrous reads the firmware's current state when the
  page opens, and the controls stay disabled on models that do not implement it.
- The state is validated before it is trusted. A firmware that answers something other than on or
  off is treated as not implementing the feature, rather than that value being read as "off" and
  offering a toggle that cannot work.

### Other Nitro models

- Nothing is keyed to a specific model. At startup Nitrous reads what the laptop reports ΓÇö its
  Acer SMBIOS tables, which Acer WMI methods exist, whether the embedded controller has a HID
  interface and which modes it lists ΓÇö and picks the mode channel from that. An unlisted Nitro
  that follows the same protocol works without a new build.
- The only per-model data is a short list of firmware quirks that cannot be probed; a model that
  is not on it gets the plain protocol.

### GPU page

- The GPU page reports the power limit the laptop is enforcing (`draw / enforced / max W`) and
  never writes it directly.

### Dashboard

- Disabled toggles now look disabled. The toggle style had no disabled state, so a control the
  model does not support looked exactly like one that worked. The same style is used by CPU Power,
  Battery Limit, USB Charging and GPU clock ownership.
- A disabled control's tooltip now appears. WPF hides tooltips on disabled elements by default, so
  the explanation of why a control was unavailable could never be read.
- The Dashboard scrolls. The custom fan section was cut off at the bottom on shorter windows.
- The `KB` page scrolls too, for the same reason, and hotkey text is centred in its box instead of
  sitting against the left edge.
- The GPU temperature no longer shows `--` while the GPU is awake: the embedded controller reports
  0 on some models, so the driver's reading is used instead when it is available.

### Display

- The refresh rate is a list of the rates the display actually advertises, instead of Auto / 60 /
  Max. `Auto` uses the rate you chose on AC and 60 Hz on battery.
- The current rate is shown next to the `DISPLAY` label, so the choice and what the screen is
  actually running are both visible.
- **LCD Overdrive** on the Dashboard, beside the refresh controls. Overdrive is a property of the
  panel rather than the model, so the control only appears where the firmware reports the panel
  supports it; where it does not, the whole row is collapsed instead of leaving a gap.
- Refresh and overdrive are two rows in one card, each with its label on the left and its control on
  the right, and the selected rate is centred in the list rather than pinned to the left edge.
- A panel without overdrive can answer the same `0` as a panel that has it switched off, so the
  change is read back after it is written. A panel that ignores the write is hidden with a log
  entry instead of looking like it worked.

### Diagnostics

- New **EC PROBE** button writes a read-only report of this machine's Acer firmware: the WMI
  classes and method IDs it exposes, the EC HID interface and the modes it lists, the SMBIOS
  gaming version, the sensor addresses Nitrous uses, and the power-mode and supported-profile
  registers. It never calls a `Set*` method. Reports are saved to
  `%LOCALAPPDATA%\Nitrous\ec-probe`, so a report from another Nitro model can be compared against
  one that is known to work.
- The probe also reports the keyboard backlight: the SMBIOS colour count, whether the EC HID and
  ENEK interfaces are present, and a read-only sweep of `GetGamingKBBacklight`. That is what shows
  whether a model without the EC interface can still set the auto-off over WMI.
- **Export Diagnostics** now includes the newest EC probe report. It is written to a separate
  folder, so it used to be missing from the bundle even when someone had just run it.
- The probe covers more of the machine: the fan behaviour, speed and table, the battery interface,
  every Acer-vendor HID interface by usage page, and the keyboard auto-off state. Both
  `APGeAction` and `AcerGenericMethod` are swept, because they are separate function spaces and
  reading the wrong one answers `0xE1` for everything.
- Every call records why it failed rather than reporting a silent null, and answers that are byte
  arrays are reported as hex instead of being dropped.

## 0.8.5 - 2026-10-27

### GPU page

- `RESET` and `APPLY` share one compact action row inside `CLOCK OFFSETS`.
- Offsets are positive-only: core `0..300 MHz`, memory `0..1000 MHz`, displayed with a `+`
  prefix. Negative values saved by older versions are clamped to `0` on first run.
- A sleeping dGPU is detected from its Windows power state instead of by probing it, so Nitrous
  no longer wakes the GPU just to find out that it is asleep.
- The clock watchdog skips its check when the active profile is `0/0` and Nitrous owns no
  offset. On battery this removes the wake that used to happen every two minutes.
- `Deep GPU Telemetry` polls only while the GPU page is open, and an asleep GPU shows `Asleep`
  instead of stale readings. It still turns itself off automatically if the GPU is powered but
  NVML stops responding.

### Hotkeys

- Editing a hotkey on the `KB` page applies immediately. The "restart required" note is gone.

### Dashboard

- Pressing the dashboard hotkey repeatedly now reuses one window instead of opening several.
- A saved off-screen position is recovered onto the primary display, including when a monitor
  is unplugged. New **Reset Dashboard Position** tray item.

## 0.8.4 - 2026-10-25

### Lower idle memory

- NVML is now initialized on demand instead of at tray startup. The tray no longer maps the
  ~23 MB NVIDIA driver image just to sit in the notification area; it loads only when the fan
  curve engine needs the NVML temperature fallback or the dashboard requests deep telemetry.
- Shutdown and re-initialization paths only touch NVML when this process actually loaded it,
  so a normal tray exit cannot pull the driver in by accident.
- The dashboard's software-rendering setup moved out of tray startup. Starting the tray no
  longer creates a WPF composition device for a process that only hosts a WinForms tray icon.
- Net effect: the idle tray process is roughly 20 MB smaller in private commit and working
  set. Dashboard launch is unchanged or slightly faster, because NVML initialization now runs
  off the UI thread alongside the existing telemetry polling.
- Per-profile GPU offsets, the 0.8.3 clock-ownership switch, the 30-second watchdog, and the
  diagnostics bundle behave exactly as before.

### Redesigned power-profile overlay

- The hotkey toast is a new card: Nitrous logo tile, `POWER PROFILE` caption, the profile name,
  a per-profile accent bar, and a status dot. Quiet is green, Balanced blue, Performance amber,
  Turbo red.
- The card is rendered once per profile and resolution into a layered-window surface, then
  animated by changing only its opacity and position. There is no per-frame text rendering and
  no WPF dependency, so the overlay does not hitch the tray or the foreground app.
- The overlay scales with the monitor's DPI and clamps to roughly 30% of the work area width on
  small panels. It appears on the monitor that owns the active window, falling back to the
  monitor under the cursor.
- A second profile change while the card is visible swaps the content in place instead of
  flashing the toast out and back in.
- The toast is shown as soon as the hotkey saves the new profile, before the WMI/NVAPI writes
  finish, so a sleeping dGPU no longer delays it. Surfaces are pre-warmed after boot, so the
  first hotkey does not pay the first-render cost.
- The app declares per-monitor V2 DPI awareness, and the overlay respects the reduced-motion
  setting by skipping the fade when UI animations are disabled.

## 0.8.3 - 2026-10-24

### GPU clock ownership is now opt-in

Nitrous no longer writes GPU clocks unless you explicitly allow it.

- Added a **Nitrous manages GPU clocks** switch in the `CLOCK OFFSETS` card on the GPU page.
- The switch defaults to **off**, including for existing installs. Updating to 0.8.3
  immediately stops every automatic GPU write. The state is not inferred from existing
  per-profile offsets, so a machine that was tuned by another tool is not taken over by an
  update.
- While the switch is off, Nitrous is read-only for GPU clocks. Startup, power-profile
  cards, the power-cycle hotkey, AC/DC switching, resume, the 30-second watchdog, **APPLY**,
  and **RESET** all skip the write. Telemetry, current clocks, temperatures, and the live
  offset readout keep working.
- While the switch is on, Nitrous owns the P0 core and memory offsets:
  - saved per-profile offsets apply at startup, profile switches, hotkeys, AC/DC changes,
    resume, and every watchdog tick;
  - a profile saved as `0/0` means stock clocks for that profile and does write `0/0`;
  - **APPLY** saves and verifies the current values, and **RESET** returns the active profile
    to stock.
- The switch is enforced inside `NvidiaGpuManager`, not only in the UI, so no call site can
  bypass it. The tray reads the shared setting on every watchdog tick, so toggling it in the
  dashboard takes effect immediately without restarting Nitrous.
- The sliders, offset boxes, **APPLY**, and **RESET** are disabled while the switch is off,
  and the card states that the displayed offsets come from the driver rather than Nitrous.

### Warnings and safe cleanup

- Enabling the switch now explains exactly what Nitrous will write, that `0/0` returns the
  GPU to stock, and that Nitrous cannot preserve another tool's voltage/frequency curve.
- If `MSIAfterburner.exe` is running, the warning is stronger: two tools writing GPU clocks
  will fight and an mV curve may reset. It asks you to set Afterburner's Core and Memory
  sliders to `0` or to close Afterburner first. If Afterburner is registered to start with
  Windows, the warning says so too.
- Turning the switch off never writes by itself. If Nitrous recorded the offsets it last
  applied, it offers to clear them once:
  - **Yes** writes `0/0` only when the GPU still reports exactly the offsets Nitrous applied;
  - **No** leaves the current offsets alone;
  - **Cancel** keeps GPU clock management on.
- Nitrous records those offsets (`GpuLastAppliedCore` / `GpuLastAppliedMemory`) after a
  verified write, so the cleanup offer cannot clobber a value another tool has since set.

### Diagnostics

- Every GPU clock write now logs its trigger (`startup`, `startup-apply`, `profile-click`,
  `hotkey`, `acdc`, `watchdog`, `apply`, `reset`, `enable`, `disable-clear`) together with the
  previous and new offsets.
- `gpu.json` and `summary.txt` now report `gpuClockManagement`, `lastAppliedOffsets`,
  `voltageOverridesDetected`, `baseVoltageCount`, `generalVoltageCount`, and the P0 base
  voltage entries with their editable flag and mV values.
- A clock write that sees an external voltage override logs a warning, so the next
  diagnostics bundle shows whether Nitrous wrote while an Afterburner curve was active.

### Behavior changes to be aware of

- An update no longer applies your saved GPU offsets until you turn the switch on. If you
  rely on Nitrous for per-profile OC, enable it once on the GPU page.
- With the switch on, switching to a `0/0` profile now returns the GPU to stock clocks.
  Previously an automatic path could leave the previous profile's offset in place.
- Startup applies a stock profile only when management is on, and a mid-boot disable stops
  the retry loop instead of writing once the driver appears.
- Fan curves, CPU power limits, processor boost, charge limit, refresh switching, Acer
  service control, hotkeys, OSD, and the updater are unchanged.

### Notes

- Nitrous still writes only P0 Graphics and Memory clock deltas through a sparse
  `SetPstates20` request and never writes voltage. It cannot preserve an external mV curve,
  so Nitrous and MSI Afterburner should not both apply GPU clocks.
- Offsets remain per power profile. `0/0` is a real value when management is on, not a
  "leave whatever is there" placeholder.

## 0.8.2 - 2026-10-24

- Battery charge and discharge telemetry now refresh after resume and AC/DC changes instead
  of showing a stale rate.
- The Acer background services page clearly reports when no restore baseline exists, instead
  of implying that a restore is available.

## 0.8.1 - 2026-10-24

- Added processor boost mode selection with separate AC and battery modes, original-value
  backups, and restore.
- Added Acer background service controls with an exact start-type and running-state backup
  and a matching restore.

## 0.8.0 - 2026-10-23

- Added a redacted diagnostics export from **Settings ΓåÆ Diagnostics**.
- Restored the power-profile OSD and hardened hardware, ACPI, and GPU state handling.
- Built-in GPU profile defaults are stock `0/0`, and legacy automatic offsets are migrated
  to `0/0` on first run.
