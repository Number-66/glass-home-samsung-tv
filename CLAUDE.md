# CLAUDE.md: Andrew's copy of Glass Home (Samsung TV)

Copy of [victaphu/glass-home-samsung-tv](https://github.com/victaphu/glass-home-samsung-tv) (MIT).
Apple TV-style home screen for Samsung Tizen TVs, sideloaded in Developer Mode.
Keep the LICENSE and CREDITS.md intact. Never push to the upstream repo.

## Who you're working with (Andrew)
- Informal, technical, direct. Short bullets, no fluff, no corporate-speak.
- Answer the literal question first, context after.
- Push back on bad ideas. Say so if something is a dead end.
- Ask before assuming. Don't dive in until you understand the whole situation.
- Verify sources and claims before presenting them as fact. This repo's README
  is one dev's report from one TV, so don't treat it as gospel.
- He goes off on tangents. Pull him back to the goal.
- AutoHotkey, if it ever comes up: v1 unless told otherwise.

## Environment
- Andrew is on **Windows**. `docs/INSTALL.md` was only run on a Mac (`ipconfig getifaddr en0`,
  `sips`, `.sh` scripts). Expect to adapt: use PowerShell or WSL/Git Bash, `ipconfig` for the PC's IP,
  Samsung's Windows Tizen CLI installer instead of the macOS `.bin`.
- Smart home stack: Hubitat hub, SmartThings (TV hub), Hue Bridge Pro, UniFi network with segmented
  VLANs. Check whether the TV is on an IoT VLAN. The PC must reach the TV on ports 8001/8002 (control API)
  and the TV must reach the PC for Developer Mode installs.
- Dev tools need Node 22 and a Chromium-based browser.

## FIRST THING: find out the goal and TV before touching anything
Ask Andrew, then wait:
1. TV model number + year (Settings > Support > About This TV). Tizen version decides what works.
2. The goal: fix something broken / run this custom launcher / hook the TV into Hubitat or SmartThings?
3. If it's a fix: exact symptom + error text, what he already tried, what changed right before it broke.

Do not start the Developer Mode / certificate process (about an hour, needs a Samsung account) until he
confirms the launcher is what he actually wants. If he only wants fewer ads/tracking, point him at
`docs/SAMSUNG-SETTINGS.md` first (about 10 minutes).

## How the original was built (the loop to copy)
```
Claude Code --WebSocket :8002--> TV   (tools/tvremote.py: keys, hold, text, status)
      ^                           |
      +-- phone camera <-- sees --+   (tools/phonecam.py: TVs can't screenshot)
```
- Act, look, adjust. Don't guess what's on screen. Get a camera frame or ask Andrew.
- `TV_IP=<ip> python tools/tvremote.py pair` shows an Allow prompt on the TV once. Token goes in
  `tools/state/` (git-ignored). Never commit it.
- `tools/wake.py` needs "Power On with Mobile" enabled on the TV.
- `phonecam.py` exposes a page to the whole LAN while it runs. Stop it when done.

## Repo map
- `app/` is the TV web app (plain HTML/CSS/JS). `app/js/config.js` holds the settings.
- `app/service/` is a Node service on the TV (photo upload website, port 8090). Code stays within the Node 12 API
  (`node dev/lint-node12.js` enforces it).
- `scripts/build.sh`, `scripts/install.sh`, `scripts/tizen-env.example.sh` are the Mac/Linux build + install path.
- `dev/` is local preview and tests: `node dev/run.js --demo`, plus `dev/test-*.js`, `dev/ui-test.js`.
- `tools/` is the Python remote, phone camera and Wake-on-LAN.
- `docs/INSTALL.md` is the install walkthrough. `docs/SAMSUNG-SETTINGS.md` covers ACR/ads/autoplay settings.

## Known limits (from the README, so don't promise otherwise)
- It does NOT replace Samsung's Home. Workaround: pin it first, "Start with Smart Hub Home" off,
  "Autorun Last App" on.
- A signed package only installs on the TV its certificate was made for. There is no shared download.
- Big Tizen OS upgrades can delete sideloaded apps. Reinstall the same way.
- Photo site is plain HTTP on the LAN. Fine at home, don't expose it.
- Website shortcuts are untested on a real TV. YouTube deep links behaved differently per method.
- 2018 TVs (Tizen 4): `photoService: false`, `smartHubLaunch: true`.

## Safety rules
- Never commit: `app/js/local-key.js`, `scripts/tizen-env.sh`, `*.p12`, `*.pwd`, `tools/state/`, anything cert-related.
- Confirm before anything that changes the TV (settings, Developer Mode, installs, factory-level stuff).
- Run `dev/` tests after code changes. Don't claim it works on the TV unless it was seen working on the TV.
- Commit on a branch, not `main`, and keep commits small.
