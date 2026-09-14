---
icon: material/gamepad
hide:
  - tags
tags:
  - romm
saltbox_automation:
  app_links:
    - name: Manual
      url: https://docs.romm.app
      type: documentation
      purpose: manual
    - name: Releases
      url: https://github.com/rommapp/romm/pkgs/container/romm
      type: github
      purpose: release
    - name: Community
      url: https://discord.gg/romm
      type: discord
      purpose: community
  project_description:
    name: RomM
    summary: |-
      a self-hosted game library manager that organises your ROMs, ISOs, and BIOS files and scrapes metadata for them.
    link: https://romm.app
---

<!-- BEGIN SALTBOX MANAGED OVERVIEW SECTION -->
<!-- This section is managed by sb-docs - DO NOT EDIT MANUALLY -->
# RomM

## Overview

[RomM](https://romm.app) is a self-hosted game library manager that organises your ROMs, ISOs, and BIOS files and scrapes metadata for them.

<div class="grid grid--buttons" markdown data-search-exclude>

[:fontawesome-solid-book-open:**Manual**](https://docs.romm.app){ .md-button .md-button--stretch }

[:fontawesome-brands-github:**Releases**](https://github.com/rommapp/romm/pkgs/container/romm){ .md-button .md-button--stretch }

[:fontawesome-brands-discord:**Community**](https://discord.gg/romm){ .md-button .md-button--stretch }

</div>

---
<!-- END SALTBOX MANAGED OVERVIEW SECTION -->

## Pre-deployment

### Library layout

RomM expects your games under a library root, organised as `roms/<platform>/<game>` and BIOS/firmware under `bios/<platform>/`. In this role the library defaults to `Media/Games` under your local library path and is mounted read-write into RomM. Use folder names that match RomM's [platform slugs](https://docs.romm.app/latest/platforms/supported-platforms/) - for example `roms/snes/` and `bios/amiga/`.

### Database

RomM uses a MariaDB backend. The role deploys a dedicated `romm-mariadb` instance automatically.

### Metadata providers

RomM enriches your library with metadata and artwork from external services. [Hasheous](https://hasheous.app) is enabled by default; the rest are opt-in and wired through environment variables. Add the keys you use to `romm_role_docker_envs_custom` in your inventory, then redeploy:

```yaml
# host_vars/localhost.yml
romm_role_docker_envs_custom:
  IGDB_CLIENT_ID: "your-id"
  IGDB_CLIENT_SECRET: "your-secret"
  MOBYGAMES_API_KEY: "your-key"
  SCREENSCRAPER_USER: "your-user"
  SCREENSCRAPER_PASSWORD: "your-pass"
  RETROACHIEVEMENTS_API_KEY: "your-key"
  STEAMGRIDDB_API_KEY: "your-key"
  LAUNCHBOX_API_ENABLED: "true"
  PLAYMATCH_API_ENABLED: "true"
  FLASHPOINT_API_ENABLED: "true"
  HLTB_API_ENABLED: "true"
```

See RomM's [Metadata Providers](https://docs.romm.app/latest/getting-started/metadata-providers/) docs for what each service provides and where to get its keys.

??? question "Want higher-performance streaming than RomM's built-in player?"

    RomM already plays many systems in the browser through its built-in [EmulatorJS](https://docs.romm.app/latest/using/in-browser-play/emulatorjs/) player - no extra setup needed. [Webstation](webstation.md) is an optional add-on that streams *full native emulators on your GPU* (Dolphin, PCSX2, RPCS3, Xenia, Yuzu/Eden, and more) for the demanding systems EmulatorJS can't run well or at all - GameCube/Wii, PS2, PS3, Xbox, and Switch among them - with much higher performance and fidelity. It needs RomM 5.3.0-alpha or newer. See the [Webstation](webstation.md) page for what it adds and how to deploy it.

    Enabling streaming is done in RomM itself (Settings → Library Management), not by the Webstation role - see the [Webstation](webstation.md) page for the streaming block to paste in.

## Deployment

```shell
sb install romm
```

This installs the current stable RomM release (the `latest` image tag) with its MariaDB backend.

??? warning "Stable vs alpha"

    Browser streaming via Webstation requires RomM **5.3.0-alpha or newer**, which is on the alpha image tag - not the default `latest`. To use streaming, pin the alpha before deploying, either on its own or by deploying Webstation (which deploys RomM for you):

    ```yaml
    # in your inventory (host_vars/localhost.yml)
    romm_role_docker_image_tag: "5.3.0-alpha.1"
    ```

    ```shell
    # RomM on the alpha tag:
    sb install romm

    # ...or let the Webstation role deploy RomM on the alpha tag for you:
    sb install sandbox-webstation
    ```

    The Webstation role refuses to run against an older RomM and tells you which tag to set.

## Usage

1. Visit <https://romm.iYOUR_DOMAIN_NAMEi> and sign in (the first account created becomes the admin).
2. Add your library: point RomM at an existing folder, or upload ROMs from the UI.
3. Run a scan so RomM fetches metadata and artwork.
4. Browse your collection, search, and build collections.

To stream native emulators (higher performance than the built-in EmulatorJS player), see [Webstation](webstation.md).

<!-- BEGIN SALTBOX MANAGED VARIABLES SECTION -->
<!-- This section is managed by sb-docs - DO NOT EDIT MANUALLY -->
<!-- END SALTBOX MANAGED VARIABLES SECTION -->
