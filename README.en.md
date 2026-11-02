# Z·SWITCH (zcode-switch)

[简体中文](README.md) ｜ **English**

A Tauri 2 desktop tool for one-click switching between multiple ZCode accounts, with live quota display. Only the login identity changes — projects, sessions, settings and plugins all stay untouched.

![screenshot](docs/screenshot.png)

## Features

- **Save / switch accounts**: one-click login switching; the current login is auto-preserved before any switch — accounts are never lost
- **Add accounts**: OAuth login for new accounts inside the tool (BigModel / z.ai entries), never touching the current login
- **Quota display**: inline plan quota and reset time per account row, multi-plan grouping
- **Claim promotions**: one-click claim for eligible promotions; "Auto claim" toggle (off by default) checks and claims periodically — manual actions take priority
- **Import / export**: exports plaintext `.json` by default (no password; switchable to encrypted `.zsb` in Settings); imports both `.json` and legacy encrypted `.zsb` (password required for `.zsb`)
- **Log out**: saves the current login to the library first, then clears the ZCode login state — available in both GUI and CLI
- **Bilingual UI (中文 / English)**: one-click switch in Settings — main window, tray, error messages and CLI output all covered; first run follows your OS language
- **Tray / autostart / CLI automation**

## Security Design

- **Local first**: all data stays on your machine; no telemetry, no remote storage; quota queries go directly to official endpoints
- **Optional export encryption**: plaintext `.json` export is the default and contains login credentials — keep the file safe; switch to encrypted `.zsb` (PBKDF2-HMAC-SHA256 100k iterations + AES-256-GCM) in Settings
- **WebView CSP**: the `script-src` baseline is `'self'`; minimal third-party script/image sources are allowed for the official promotion web component. UI events do not rely on dynamic execution — they go through a whitelist-based dispatcher
- **No account loss**: the current login is auto-preserved before switching if not yet saved; file writes go through a temp file + atomic rename
- **Path traversal protection**: account id whitelist (`[A-Za-z0-9-]`); delete/read cannot escape the account store directory
- **Legacy encrypted bundle compatibility**: encrypted `.zsb` (PBKDF2-HMAC-SHA256 100k iterations + AES-256-GCM, random salt/nonce) can still be imported; a wrong password simply fails
- **Credentials decrypted locally only**: decryption is used solely to display username/email

## CLI

```
zcode-switch.exe --cli state|list
zcode-switch.exe --cli quota [--id <account-id>]
zcode-switch.exe --cli claim-preview [--id <account-id>]
zcode-switch.exe --cli capture [--name <name>]
zcode-switch.exe --cli switch --id <id> [--force] [--restart|--no-restart] [--hot <bool>|--no-hot]
zcode-switch.exe --cli logout [--restart|--no-restart]
zcode-switch.exe --cli kill
zcode-switch.exe --cli export --id <id> --out <a.json> [--plain|--encrypt]
zcode-switch.exe --cli export-all --out <all.json> [--plain|--encrypt]
zcode-switch.exe --cli import --file <file.json|file.zsb>
zcode-switch.exe --cli rename|delete|update|behavior|setpath|launch
zcode-switch.exe --cli --lang en state              # English output (--lang takes a space-separated value, works anywhere in the command line; defaults to the GUI/system language)
```

CLI export follows the "Plaintext export" setting by default; override with `--plain` / `--encrypt`. CLI password (only for `--encrypt` export or importing a legacy encrypted `.zsb`): prefer the `ZSW_PASSWORD` environment variable (keeps it out of process lists and command history); `--password <password>` also works.

## FAQ

### macOS asks for Microphone / Accessibility / Screen Recording permission?

Deny all of them — nothing breaks.
The embedded pages (such as the login page) are rendered by the system WebView, which relays their requests as system permission prompts attributed to the app; neither the app itself nor its embedded pages use any of these three capabilities.

## Build

```bash
npm install
npm run tauri dev      # development (HMR)
npm run tauri build    # NSIS installer
```

Windows-first (path detection / process management / tray are all Win32 semantics).

## License

[MIT](./LICENSE)
