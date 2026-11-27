# FileZilla impersonation → Base dead-drop → Electron loader

Multi-brand fake-software campaign. Lures load an obfuscated `/filedrop.js`
dropper, resolve a backend through a smart contract on Base, complete an
ECDH P-256 handshake, and deliver a 131 MB ZIP containing a signed Twake
Desktop build with a malicious `app.asar`.

Stage 2 is a PLAC/IAOB bundle named `volume_handler.json`.

Original discovery: <https://x.com/inf0stache/status/2125653864984772753>

## Chain

1. The user visits a fake software site (FileZilla, Malwarebytes, VirtualDJ,
   TwinkleTray, CrystalMark, and others) and loads `/filedrop.js`.
2. The loader calls `backend()` (`0x099e4133`) on a Base contract through a
   rotating list of public RPCs.
3. The contract returns a backend URL. The loader performs an ECDH P-256
   handshake against `POST /v1/hs` and requests a ticket from `POST /v1/t`.
4. A per-ticket subdomain of `secure-bot.club` serves `GET /v1/d?t=<ticket>`:
   a 131 MB ZIP with `Inst.app_*.exe` (signed Twake Desktop) and a malicious
   `resources/app.asar`.
5. The Electron loader persists via `setLoginItemSettings`, beacons to
   `github.desktop.ug/api/` every 180 seconds, and executes `task.e` with
   `eval` or writes and runs `task.files`.
6. Stage 2 (`volume_handler.json`) is a PLAC/IAOB container with `ti64`/`ti`
   images, `CUSTOMINJECT`, and UAC/WD/shell modules.

## Samples (SHA-256)

### Stage 1 — loader and delivery

```
95c1a51f44ee290c9d08125dd7385f6be79562f73ebd47f9561dd3e1ef359d9d filedrop.js (Build A)
2a499cacc9cad67e981675ab67e5ef002b093712ca5a69ef1c895b2401915603 filedrop.js (Build B)
897255a6215222b53f2493e2131a90c29ba63e37db7309b33c15533c45319b1e latest-version_*.zip
aacb94e5c02431d9e6c9b1230b0c688299bf2a9a35416b27a11d7f1228516b26 resources/app.asar
```

`Inst.app_*.exe` is a validly signed Twake Desktop host and is not listed as
malware. The malicious code is in `app.asar`.

### Stage 2 — malicious JSON bundle

```
9110b59c1eb0f0de0138721a3adb59247937916c4a967439f8e693b68d947257 volume_handler.json
a51c66bf3209716de0964f2d01639592dffdbed163e0ff9a04fab3bd381847fc decoded PLAC/IAOB bundle
```

Only `volume_handler.json` (and its decoded bundle) is treated as the malicious
stage-2 component. The accompanying `eventprime.dat` file and the signed iTop,
SQLite and Microsoft runtime files are not listed as malware samples.

## Network indicators

### Domains

```
file-zilla[.]com
malware-bytes[.]net
twinkletraysoftware[.]com
virtualdjsoftware[.]com
crystalmarksoftware[.]com
crystalmarksoftware[.]org
crystalmarksoftware[.]net
crystalmarksoftware[.]pro
crystalmarksoftware[.]app
wisecare365[.]org
projectobs[.]org
notepad-plus-plus[.]app
furmarksoftware[.]org
inkscapesoftware[.]com
getfancontrol[.]org
windhawksoftware[.]com
privazer[.]app
get-sharex[.]com
edcv[.]com[.]au
ficoso[.]click
secure-link[.]club
secure-bot[.]club
github[.]desktop[.]ug
178.16.52[.]174
```

`crystalmarksoftware[.]com` no longer resolves; `crystalmarksoftware[.]org`
redirects to `.pro`; `ficoso[.]click` returns HTTP 522; `edcv[.]com[.]au`
returns HTTP 404. They are retained for historical hunting.

### URLs

```
hxxps://secure-bot[.]club/v1/hs
hxxps://secure-bot[.]club/v1/t
hxxps://secure-bot[.]club/v1/d?t=
hxxps://<ticket>.secure-bot[.]club/
hxxps://github[.]desktop[.]ug/api/
hxxps://ficoso[.]click/v1/hs
```

### Base contracts

```
0x4359890055Dc647118fc7c5b0082eABd757C40C8
0xe55B08Dced66B6Bf99924eC18c2B476BB9Df40FB
0x58af855c76b9281375f27e355ec97fd89d9da625
0x57a046432f9478cee329969f46fdb1ec017961b8
```

The first two are the resolver contracts (`backend()` selector
`0x099e4133`); the last two are their owner EOAs. The contracts expose an
owner-only string setter (`0x21fb884d`) and were repointed between backends.

### Resolver RPCs (public, not malicious)

```
https://mainnet.base.org
https://base.llamarpc.com
https://base.publicnode.com
https://1rpc.io/base
https://base.drpc.org
https://base-mainnet.public.blastapi.io
https://base.gateway.tenderly.co
```

## File names

```
filedrop.js
volume_handler.json
eventprime.dat
latest-version_*.zip
Inst.app_*.exe
readme.txt
daemon.log
```

## Behavioural notes

- `/download` auto-invokes `FileDrop.download()`; no user click is required.
- Victims store an identifier in `%APPDATA%\daemon.log`.
- `readme.txt` beside the Electron EXE supplies an affiliate slot in the form
  `<name>-<company>`.
- The loader beacons to `github.desktop.ug/api/` with
  `[id, COMPUTERNAME, USERNAME, slot_name, slot_company]` every 180 seconds.
- `task.e` is executed with `eval`; `task.files` are base64-decoded into
  `%TEMP%\<epoch>\` and the first `.exe` is executed.
