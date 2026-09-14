---
icon: material/controller
hide:
  - tags
tags:
  - webstation
saltbox_automation:
  app_links:
    - name: Manual
      url: https://docs.romm.app/latest/using/emulator-streaming-migration/
      type: documentation
      purpose: manual
    - name: Releases
      url: https://github.com/linuxserver/docker-webstation/pkgs/container/webstation
      type: github
      purpose: release
    - name: Community
      url: https://linuxserver.io/discord
      type: discord
      purpose: community
  project_description:
    name: Webstation
    summary: |-
      a browser-based game streaming frontend (built on Selkies) that runs emulated and native games from your RomM library and streams them to your browser.
    link: https://docs.romm.app/latest/using/emulator-streaming-migration/
---

<!-- BEGIN SALTBOX MANAGED OVERVIEW SECTION -->
<!-- This section is managed by sb-docs - DO NOT EDIT MANUALLY -->
# Webstation

## Overview

[Webstation](https://docs.romm.app/latest/using/emulator-streaming-migration/) is a browser-based game streaming frontend (built on Selkies) that runs emulated and native games from your RomM library and streams them to your browser. It replaces the old one-container-per-emulator streaming setup with a single container that can serve every platform RomM supports.

<div class="grid grid--buttons" markdown data-search-exclude>

[:fontawesome-solid-book-open:**Manual**](https://docs.romm.app/latest/using/emulator-streaming-migration/){ .md-button .md-button--stretch }

[:fontawesome-brands-github:**Releases**](https://github.com/linuxserver/docker-webstation/pkgs/container/webstation){ .md-button .md-button--stretch }

[:fontawesome-brands-discord:**Community**](https://linuxserver.io/discord){ .md-button .md-button--stretch }

</div>

---
<!-- END SALTBOX MANAGED OVERVIEW SECTION -->

??? info "How is this different from RomM's built-in player?"

    RomM already plays many systems in the browser through its built-in EmulatorJS player (a WebAssembly emulator, no extra setup). Webstation instead streams *full native emulators running on your GPU* - Dolphin, PCSX2, RPCS3, Xenia, Yuzu/Eden, and the rest of RetroArch's cores. Use it for the heavier systems EmulatorJS can't run well or at all: GameCube/Wii, PS2, PS3, Xbox, and Switch, where you want real performance and fidelity.

## Pre-deployment

Webstation is a streaming *companion* to [RomM](romm.md) - it does not manage a library of its own. The role depends on RomM and deploys it automatically unless you disable `webstation_role_dependency_romm`.

### Requirements

- **An x86-64 host.** The LinuxServer Webstation image is published for `amd64` only; arm64 is not supported.
- **A GPU.** Webstation runs in Wayland mode and needs a GPU (Intel or Nvidia) with `/dev/dri` exposed. Without hardware rendering, games are not playable. The role enables the render device automatically when present.
- **RomM 5.3.0 or newer** with streaming enabled. The `webstation` streaming protocol is new in 5.3; older RomM rejects the config and the role fails with upgrade instructions.
- **BIOS/firmware files** for the systems you want to stream. Place them in your RomM library under `bios/<platform>/` (for example `bios/amiga/` for Amiga kickstarts and `capsimg.so`). They are made available to the emulators automatically - re-run the role after adding new files.

## Deployment

```shell
sb install sandbox-webstation
```

This creates the `romm-webstation` container, a `romm-stream` DNS record, and the matching Traefik route. Webstation is self-configuring via environment variables, and it also writes the streaming block into RomM's `config.yml` for you - no manual RomM configuration is required (RomM's UI exposes no streaming settings).

## Enable streaming (managed by the role)

The `webstation` role automatically injects a `streaming` block into RomM's `config.yml` (delimited by `# BEGIN/END ANSIBLE MANAGED STREAMING (webstation role)`). The block points RomM at this container using the same `BROKER_SECRET` the webstation container runs with, so the two stay in sync across reruns. RomM lists every platform the role supports (see `webstation_role_platforms`), each mapped to its emulator. A representative view of what the role writes:

```yaml
streaming:
  enabled: true
  containers:
    - protocol: webstation
      host: "https://romm-stream.YOUR_DOMAIN_NAME"
      subfolder: /streaming
      broker_host: "http://romm-webstation:3000"
      broker_secret: "<auto-managed, matches BROKER_SECRET>"
      label: Webstation
      library_path: /romm/library
      platforms:
        3do:
          emulator: retroarch
          label: 3DO
        snes:
          emulator: retroarch
          label: SNES
        ps2:
          emulator: pcsx2
          label: PS2
        ngc:
          emulator: dolphin
          label: GameCube
```

- `broker_secret` is the webstation container's `BROKER_SECRET` (read it with `docker exec romm-webstation printenv BROKER_SECRET`); the role keeps RomM's copy equal to it.
- `broker_host` is the server-to-broker API on the docker network; `host` is the browser-facing URL (no `/streaming` suffix - that is `subfolder`).
- The full block is generated from `webstation_role_platforms`; edit that list in host_vars to add or remove platforms. See RomM's [streaming reference](https://docs.romm.app/latest/reference/configuration-file/#streaming) for every key.

Because the block is marked and managed, **do not hand-edit it** - change `webstation_role_platforms` (or re-run `sb install sandbox-webstation`) instead.

## Usage

1. Open <https://romm.iYOUR_DOMAIN_NAMEi> and sign in through the stack SSO.
2. Browse to a game and choose **Play**. RomM brokers a streaming session and opens the Selkies client at <https://romm-stream.iYOUR_DOMAIN_NAMEi/streaming>.
3. Play with keyboard and mouse, or plug in a gamepad - most emulators pick it up automatically. Use the on-screen menu for fullscreen and control mapping.

Every platform RomM supports appears as a Play option; the Webstation container claims a free emulator instance for each launch. Desktop sessions (the full emulator desktop) launch the same way from RomM.

??? tip "Performance"

    Streaming is GPU-bound. If a game is choppy, lower the in-game resolution or close other GPU workloads on the host. A black screen on launch almost always means missing BIOS for that platform or no GPU available to the container - check both before troubleshooting the network.

??? warning "Direct LAN access is off by default"

    The underlying image also serves its desktop GUI directly on ports `3000`/`3001` over a self-signed certificate. That is **disabled by default** in this role because the GUI has no built-in authentication and includes a terminal.

    To enable it on a trusted local network only, set in your inventory:

    ```yaml
    webstation_role_direct_access_enabled: true
    ```

<!-- BEGIN SALTBOX MANAGED VARIABLES SECTION -->
<!-- This section is managed by sb-docs - DO NOT EDIT MANUALLY -->
<!-- END SALTBOX MANAGED VARIABLES SECTION -->
