<a name="readme-top"></a>

<div align="center">

<img src="../logo.png" alt="LumiRealm" width="640"/>

[English](../README.md) | [한국어](readme-ko_kr.md) | **日本語** | [简体中文](readme-zh_cn.md) | [繁體中文](readme-zh_tw.md) | [Deutsch](readme-de_de.md) | [Русский](readme-ru_ru.md)

[![License](https://img.shields.io/badge/license-GPL--3.0--or--later-blue)](../LICENSE)
[![Lumiverse](https://img.shields.io/badge/Lumiverse-1.0.0%2B-blueviolet)](https://github.com/prolix-oc/Lumiverse)
[![RisuAI](https://img.shields.io/badge/RisuAI-port-9cf?logo=svelte)](https://github.com/kwaroran/Risuai)
[![TypeScript](https://img.shields.io/badge/TypeScript-5-blue?logo=typescript)](https://www.typescriptlang.org/)
[![Bun](https://img.shields.io/badge/Bun-bundle-fbf0df?logo=bun)](https://bun.sh)

</div>

---

LumiRealm は、[RisuAI](https://github.com/kwaroran/Risuai) のキャラクターカード、モジュール、ロアブックを [Lumiverse](https://github.com/prolix-oc/Lumiverse) 内でネイティブに動作させる Lumiverse 拡張機能です。RisuRealm のボットブラウザを内蔵しています。

詳しいガイドは **[Wiki](https://github.com/AMousePad/LumiRealm/wiki)** をご覧ください。

## スクリーンショット

|                  カード例                   |                       RisuRealm 検索                       |
| :------------------------------------------: | :--------------------------------------------------------: |
| ![1778064388761](../image/README/1778064388761.png) | ![1778064256839](../image/README/1778064256839.png) |

|                    ビューア                    |                     ステート                    |
| :--------------------------------------------: | :--------------------------------------------: |
| ![1778064299483](../image/README/1778064299483.png) | ![1778064443131](../image/README/1778064443131.png) |

## インストール

LumiRealm は Lumiverse の拡張機能としてインストールします。Lumiverse は **1.0.0 以上** が必要です。

1. Lumiverse のインスタンスを開きます。
2. **サイドバー → 下にスクロール → 拡張機能タブ** に移動して、次を追加します:

   ```txt
   https://github.com/AMousePad/LumiRealm
   ```
3. まずすべての権限を有効にしてください。LumiRealm の動作にはすべての権限が必要です。[なぜ?](https://github.com/AMousePad/LumiRealm/wiki/Architecture)
4. 拡張を有効にすると、サイドバーに **LumiRealm** タブが表示されます。

## コミュニティ

- **[Discord](https://github.com/AMousePad/LumiRealm/wiki/Discord)**: Lumiverse サーバーでお会いしましょう!
- **[Issues](https://github.com/AMousePad/LumiRealm/issues)**: バグ報告と機能リクエスト。
- **[Wiki](https://github.com/AMousePad/LumiRealm/wiki)**: ユーザーガイドとアーキテクチャ詳細解説。

## ライセンス

**GPL-3.0-or-later.** LumiRealm は [RisuAI](https://github.com/kwaroran/Risuai) (GPL-3.0, © 2024 Kwaroran) の派生著作物です。一部のモジュール全体 (Lua ブリッジの prelude、すべての CBS ハンドラ、`processScriptFull` の移植、モジュール/トグル DSL パーサー、ロアブックデコレータパーサー) は直接の移植です。Risu のコンパイル済み CSS バンドルはそのまま同梱されています。

<p align="right">(<a href="#readme-top">トップへ戻る</a>)</p>
