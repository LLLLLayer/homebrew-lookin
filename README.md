# Homebrew Tap for ivista-lookin

`ivista-lookin` is a standalone command-line interface for inspecting iOS apps that integrate LookinServer. It does not require the macOS Lookin.app to be installed, but the target iOS app must already include a compatible LookinServer integration.

## Install

```bash
brew install LLLLLayer/ivista-lookin/ivista-lookin
ivista-lookin --version
ivista-lookin doctor
```

Upgrade:

```bash
brew update
brew upgrade ivista-lookin
```

Uninstall:

```bash
brew uninstall ivista-lookin
```

## Requirements

1. macOS with Homebrew.
2. An iOS app running on a simulator or USB device.
3. The target iOS app must integrate LookinServer in Debug builds.
4. Do not integrate LookinServer in Release/App Store builds.

## Quick Start

List reachable apps:

```bash
ivista-lookin apps
ivista-lookin apps --json
```

Inspect a target app:

```bash
ivista-lookin tree --bundle-id com.example.demo --depth 2
ivista-lookin find --bundle-id com.example.demo UILabel --limit 10
ivista-lookin inspect --bundle-id com.example.demo --oid 123 --json
ivista-lookin attrs --bundle-id com.example.demo --oid 123
```

When multiple apps or devices match, add selectors:

```bash
ivista-lookin apps --transport usb
ivista-lookin tree --bundle-id com.example.demo --transport usb --device-id 14
ivista-lookin tree --bundle-id com.example.demo --transport simulator --port 47164
```

## Commands

```text
ivista-lookin --help
ivista-lookin --version
ivista-lookin doctor
ivista-lookin apps [--json] [--bundle-id <bundle-id>] [--transport simulator|usb] [--port <port>] [--device-id <id>]
ivista-lookin tree --bundle-id <bundle-id> [--json] [--depth N] [--filter <text>] [--oid <oid>] [--transport simulator|usb] [--port <port>] [--device-id <id>]
ivista-lookin find --bundle-id <bundle-id> <query> [--json] [--limit N] [--transport simulator|usb] [--port <port>] [--device-id <id>]
ivista-lookin find --bundle-id <bundle-id> --oid <oid> [--json] [--transport simulator|usb] [--port <port>] [--device-id <id>]
ivista-lookin inspect --bundle-id <bundle-id> --oid <oid> [--json] [--transport simulator|usb] [--port <port>] [--device-id <id>]
ivista-lookin attrs --bundle-id <bundle-id> --oid <oid> [--group <filter>] [--json] [--transport simulator|usb] [--port <port>] [--device-id <id>]
ivista-lookin screenshot --bundle-id <bundle-id> --oid <oid> --out <path> [--type group|solo] [--json] [--transport simulator|usb] [--port <port>] [--device-id <id>]
ivista-lookin export --bundle-id <bundle-id> --out <file.lookin> [--compression 0.01-1] [--json] [--transport simulator|usb] [--port <port>] [--device-id <id>]
ivista-lookin read <file.lookin> [summary|tree|find|inspect|attrs] [options]
ivista-lookin selectors --bundle-id <bundle-id> (--class <class-name> | --oid <oid>) [--with-args] [--filter <text>] [--json] [--transport simulator|usb] [--port <port>] [--device-id <id>]
ivista-lookin call --bundle-id <bundle-id> --oid <oid> --selector <selector> [--json] [--transport simulator|usb] [--port <port>] [--device-id <id>]
ivista-lookin eval --bundle-id <bundle-id> --oid <oid> <property-or-method> [--json] [--transport simulator|usb] [--port <port>] [--device-id <id>]
ivista-lookin console --bundle-id <bundle-id> --oid <oid> [--transport simulator|usb] [--port <port>] [--device-id <id>]
ivista-lookin set --bundle-id <bundle-id> --oid <oid> --attr <identifier> --value <value> [--dry-run] [--json] [--transport simulator|usb] [--port <port>] [--device-id <id>]
```

## Common Workflows

Find an object id:

```bash
ivista-lookin tree --bundle-id com.example.demo --depth 3
ivista-lookin find --bundle-id com.example.demo UIButton --json
```

Read object details:

```bash
ivista-lookin inspect --bundle-id com.example.demo --oid 123
ivista-lookin attrs --bundle-id com.example.demo --oid 123 --group frame
```

Export screenshots and snapshots:

```bash
ivista-lookin screenshot --bundle-id com.example.demo --oid 123 --out button.png
ivista-lookin export --bundle-id com.example.demo --out demo.lookin
```

Read an exported `.lookin` file offline:

```bash
ivista-lookin read demo.lookin summary
ivista-lookin read demo.lookin tree --depth 2
ivista-lookin read demo.lookin find UILabel --json
ivista-lookin read demo.lookin inspect --oid 123
ivista-lookin read demo.lookin attrs --oid 123
```

Use the console-style commands:

```bash
ivista-lookin selectors --bundle-id com.example.demo --oid 123 --filter title
ivista-lookin call --bundle-id com.example.demo --oid 123 --selector description
ivista-lookin eval --bundle-id com.example.demo --oid 123 frame --json
ivista-lookin console --bundle-id com.example.demo --oid 123
```

Modify settable attributes:

```bash
ivista-lookin attrs --bundle-id com.example.demo --oid 123 --json
ivista-lookin set --bundle-id com.example.demo --oid 123 --attr l_f_f --value "0,0,120,44" --dry-run
ivista-lookin set --bundle-id com.example.demo --oid 123 --attr l_f_f --value "0,0,120,44"
```

`set` supports built-in settable attributes and custom attributes that expose a `customSetterID`.

## JSON and Scripting

Use `--json` for machine-readable output. Warnings and errors are written to stderr; JSON is written to stdout.

```bash
ivista-lookin apps --json | jq .
ivista-lookin tree --bundle-id com.example.demo --depth 1 --json > tree.json
```

## Lookin.app Conflict

Current LookinServer versions support one client session per target app process. If macOS Lookin.app is already connected to the same iOS app, `ivista-lookin` may not see or connect to that target.

Check the status:

```bash
ivista-lookin doctor
```

If `Lookin.app` is running and online commands cannot find your app, quit the macOS Lookin.app and retry:

```bash
ivista-lookin apps --json
```

Offline commands such as `ivista-lookin read <file.lookin>` are not affected.

## Troubleshooting

No app is found:

```bash
ivista-lookin doctor
ivista-lookin apps --json
```

Then check:

1. The iOS app is running.
2. The iOS app integrates LookinServer in Debug.
3. The selected transport, port, and device id are correct.
4. macOS Lookin.app is not occupying the current LookinServer session.

Server version errors mean the target app's LookinServer protocol is outside the CLI-supported range. Upgrade the target app's LookinServer integration, or use a CLI version compatible with that app.

## Project Links

- CLI source and releases: https://github.com/LLLLLayer/ivista-lookin
- This Homebrew tap: https://github.com/LLLLLayer/homebrew-ivista-lookin
- LookinServer: https://github.com/QMUI/LookinServer
