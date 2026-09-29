<a name="readme-top"></a>

<div align="center">

<img src="../logo.png" alt="LumiRealm" width="640"/>

[English](../README.md) | [한국어](readme-ko_kr.md) | [日本語](readme-ja_jp.md) | **简体中文** | [繁體中文](readme-zh_tw.md) | [Deutsch](readme-de_de.md) | [Русский](readme-ru_ru.md)

[![License](https://img.shields.io/badge/license-GPL--3.0--or--later-blue)](../LICENSE)
[![Lumiverse](https://img.shields.io/badge/Lumiverse-1.0.0%2B-blueviolet)](https://github.com/prolix-oc/Lumiverse)
[![RisuAI](https://img.shields.io/badge/RisuAI-port-9cf?logo=svelte)](https://github.com/kwaroran/Risuai)
[![TypeScript](https://img.shields.io/badge/TypeScript-5-blue?logo=typescript)](https://www.typescriptlang.org/)
[![Bun](https://img.shields.io/badge/Bun-bundle-fbf0df?logo=bun)](https://bun.sh)

</div>

---

LumiRealm 是一个 [Lumiverse](https://github.com/prolix-oc/Lumiverse) 扩展,可以在 Lumiverse 中原生运行 [RisuAI](https://github.com/kwaroran/Risuai) 的角色卡、模块和世界书。内置 RisuRealm 机器人浏览器。

完整指南请见 **[Wiki](https://github.com/AMousePad/LumiRealm/wiki)**。

## 截图

|                  示例卡片                   |                       RisuRealm 浏览                       |
| :------------------------------------------: | :--------------------------------------------------------: |
| ![1778064388761](../image/README/1778064388761.png) | ![1778064256839](../image/README/1778064256839.png) |

|                     查看器                     |                     状态                      |
| :--------------------------------------------: | :--------------------------------------------: |
| ![1778064299483](../image/README/1778064299483.png) | ![1778064443131](../image/README/1778064443131.png) |

## 安装

LumiRealm 作为 Lumiverse 扩展安装。Lumiverse 需要 **1.0.0 或更高版本**。

1. 打开你的 Lumiverse 实例。
2. 进入 **侧边栏 → 向下滚动 → 扩展选项卡** 并添加:

   ```txt
   https://github.com/AMousePad/LumiRealm
   ```
3. 请先启用所有权限。LumiRealm 运行时全部都需要。[为什么?](https://github.com/AMousePad/LumiRealm/wiki/Architecture)
4. 启用扩展后,**LumiRealm** 标签会出现在侧边栏中。

## 社区

- **[Discord](https://github.com/AMousePad/LumiRealm/wiki/Discord)**: Lumiverse 服务器见!
- **[Issues](https://github.com/AMousePad/LumiRealm/issues)**: Bug 反馈与功能请求。
- **[Wiki](https://github.com/AMousePad/LumiRealm/wiki)**: 用户指南和架构深入说明。

## 许可证

**GPL-3.0-or-later.** LumiRealm 是 [RisuAI](https://github.com/kwaroran/Risuai)(GPL-3.0,© 2024 Kwaroran)的衍生作品。整个模块(Lua 桥接 prelude、所有 CBS 处理器、`processScriptFull` 移植、模块/开关 DSL 解析器、世界书装饰器解析器)是直接移植。Risu 编译后的 CSS 包按原样附带。

<p align="right">(<a href="#readme-top">回到顶部</a>)</p>
