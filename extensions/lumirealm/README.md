<!-- Mirror of upstream documentation. Everything below the metadata table is the original file, unmodified. -->

| Field | Value |
| --- | --- |
| Extension | LumiRealm |
| Source repository | https://github.com/AMousePad/LumiRealm |
| Original link (from Lumiverse-Extensions README) | https://github.com/AMousePad/LumiRealm/ |
| Upstream path | `README.md` |
| Retrieved | 2026-09-29 @ `main` (`e3a95c3`) |

---

<a name="readme-top"></a>

<div align="center">

<img src="logo.png" alt="LumiRealm" width="640"/>

English | [한국어](.github/readme-ko_kr.md) | [日本語](.github/readme-ja_jp.md) | [简体中文](.github/readme-zh_cn.md) | [繁體中文](.github/readme-zh_tw.md) | [Deutsch](.github/readme-de_de.md) | [Русский](.github/readme-ru_ru.md)

[![License](https://img.shields.io/badge/license-GPL--3.0--or--later-blue)](LICENSE)
[![Lumiverse](https://img.shields.io/badge/Lumiverse-1.0.0%2B-blueviolet)](https://github.com/prolix-oc/Lumiverse)
[![RisuAI](https://img.shields.io/badge/RisuAI-port-9cf?logo=svelte)](https://github.com/kwaroran/Risuai)
[![TypeScript](https://img.shields.io/badge/TypeScript-5-blue?logo=typescript)](https://www.typescriptlang.org/)
[![Bun](https://img.shields.io/badge/Bun-bundle-fbf0df?logo=bun)](https://bun.sh)

</div>

---

LumiRealm is a [Lumiverse](https://github.com/prolix-oc/Lumiverse) extension that runs [RisuAI](https://github.com/kwaroran/Risuai) character cards, modules, and lorebooks natively inside Lumiverse. Includes an inbuilt RisuRealm bot browser.

Full guide on the **[Wiki](https://github.com/AMousePad/LumiRealm/wiki)**.

## Screenshots

|                  Example Card                  |                                                    RisuRealm browse                                                    |
| :--------------------------------------------: | :---------------------------------------------------------------------------------------------------------------------: |
| ![1778064388761](image/README/1778064388761.png) | ![1778064256839](image/README/1778064256839.png) |

|                     Viewer                     |                     State                     |
| :--------------------------------------------: | :--------------------------------------------: |
| ![1778064299483](image/README/1778064299483.png) | ![1778064443131](image/README/1778064443131.png) |

## Installation

LumiRealm installs as a Lumiverse extension. Lumiverse must be at version **1.0.0 or later.**

1. Open your Lumiverse instance.
2. Go to the **Sidebar → Scroll Down → Extensions Tab** and add:

   ```txt
   https://github.com/AMousePad/LumiRealm
   ```
3. Enable all of the permissions first. LumiRealm needs them all to run. [Why?](https://github.com/AMousePad/LumiRealm/wiki/Architecture)
4. Enable the extension. The **LumiRealm** tab appears in the sidebar.

## Branches

Pick the branch that matches the Lumiverse you're running.

- **`main`** is the default. Tracks the latest released Lumiverse. Use this unless you're running Lumiverse from source on `staging`.
- **`staging`** tracks unreleased Lumiverse APIs that haven't shipped yet. Requires you to be running Lumiverse `staging` from source. Bleeding-edge features land here first. They migrate to `main` when the Lumiverse changes they depend on ship in a release.

To switch branches after installing, go to the extensions tab and use the **Branch** button on the LumiRealm entry.

Small fixes that don't depend on unreleased Lumiverse APIs land on `main` directly. Anything that depends on a Lumiverse change not yet in a release lands on `staging` first.

## Community

- **[Discord](https://github.com/AMousePad/LumiRealm/wiki/Discord)**: In the Lumiverse server!
- **[Issues](https://github.com/AMousePad/LumiRealm/issues)**: bug reports + feature requests.
- **[Wiki](https://github.com/AMousePad/LumiRealm/wiki)**: user guide + architecture deep-dive.

## License

**GPL-3.0-or-later.** LumiRealm is a derivative work of [RisuAI](https://github.com/kwaroran/Risuai) (GPL-3.0, © 2024 Kwaroran). Entire modules (Lua bridge prelude, all CBS handlers, `processScriptFull` port, module/toggle DSL parser, lorebook decorator parser) are direct ports. Risu's compiled CSS bundle ships verbatim.

<p align="right">(<a href="#readme-top">back to top</a>)</p>
