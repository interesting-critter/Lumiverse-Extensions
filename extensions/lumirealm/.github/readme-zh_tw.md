<a name="readme-top"></a>

<div align="center">

<img src="../logo.png" alt="LumiRealm" width="640"/>

[English](../README.md) | [한국어](readme-ko_kr.md) | [日本語](readme-ja_jp.md) | [简体中文](readme-zh_cn.md) | **繁體中文** | [Deutsch](readme-de_de.md) | [Русский](readme-ru_ru.md)

[![License](https://img.shields.io/badge/license-GPL--3.0--or--later-blue)](../LICENSE)
[![Lumiverse](https://img.shields.io/badge/Lumiverse-1.0.0%2B-blueviolet)](https://github.com/prolix-oc/Lumiverse)
[![RisuAI](https://img.shields.io/badge/RisuAI-port-9cf?logo=svelte)](https://github.com/kwaroran/Risuai)
[![TypeScript](https://img.shields.io/badge/TypeScript-5-blue?logo=typescript)](https://www.typescriptlang.org/)
[![Bun](https://img.shields.io/badge/Bun-bundle-fbf0df?logo=bun)](https://bun.sh)

</div>

---

LumiRealm 是一個 [Lumiverse](https://github.com/prolix-oc/Lumiverse) 擴充功能,可在 Lumiverse 中原生執行 [RisuAI](https://github.com/kwaroran/Risuai) 的角色卡、模組與世界書。內建 RisuRealm 機器人瀏覽器。

完整指南請見 **[Wiki](https://github.com/AMousePad/LumiRealm/wiki)**。

## 螢幕截圖

|                  範例卡片                   |                       RisuRealm 瀏覽                       |
| :------------------------------------------: | :--------------------------------------------------------: |
| ![1778064388761](../image/README/1778064388761.png) | ![1778064256839](../image/README/1778064256839.png) |

|                     檢視器                     |                     狀態                      |
| :--------------------------------------------: | :--------------------------------------------: |
| ![1778064299483](../image/README/1778064299483.png) | ![1778064443131](../image/README/1778064443131.png) |

## 安裝

LumiRealm 以 Lumiverse 擴充功能安裝。Lumiverse 需為 **1.0.0 以上版本**。

1. 開啟你的 Lumiverse 執行實例。
2. 進入 **側邊欄 → 向下捲動 → 擴充功能分頁** 並新增:

   ```txt
   https://github.com/AMousePad/LumiRealm
   ```
3. 請先啟用所有權限。LumiRealm 運作時全部都需要。[為什麼?](https://github.com/AMousePad/LumiRealm/wiki/Architecture)
4. 啟用擴充功能後,**LumiRealm** 分頁會出現在側邊欄中。

## 社群

- **[Discord](https://github.com/AMousePad/LumiRealm/wiki/Discord)**: Lumiverse 伺服器見!
- **[Issues](https://github.com/AMousePad/LumiRealm/issues)**: Bug 回報與功能請求。
- **[Wiki](https://github.com/AMousePad/LumiRealm/wiki)**: 使用者指南與架構深入說明。

## 授權條款

**GPL-3.0-or-later.** LumiRealm 是 [RisuAI](https://github.com/kwaroran/Risuai)(GPL-3.0,© 2024 Kwaroran)的衍生作品。整個模組(Lua 橋接 prelude、所有 CBS 處理器、`processScriptFull` 移植、模組/開關 DSL 剖析器、世界書修飾器剖析器)為直接移植。Risu 編譯後的 CSS 套件按原樣隨附。

<p align="right">(<a href="#readme-top">回到頂端</a>)</p>
