<a name="readme-top"></a>

<div align="center">

<img src="../logo.png" alt="LumiRealm" width="640"/>

[English](../README.md) | **한국어** | [日本語](readme-ja_jp.md) | [简体中文](readme-zh_cn.md) | [繁體中文](readme-zh_tw.md) | [Deutsch](readme-de_de.md) | [Русский](readme-ru_ru.md)

[![License](https://img.shields.io/badge/license-GPL--3.0--or--later-blue)](../LICENSE)
[![Lumiverse](https://img.shields.io/badge/Lumiverse-1.0.0%2B-blueviolet)](https://github.com/prolix-oc/Lumiverse)
[![RisuAI](https://img.shields.io/badge/RisuAI-port-9cf?logo=svelte)](https://github.com/kwaroran/Risuai)
[![TypeScript](https://img.shields.io/badge/TypeScript-5-blue?logo=typescript)](https://www.typescriptlang.org/)
[![Bun](https://img.shields.io/badge/Bun-bundle-fbf0df?logo=bun)](https://bun.sh)

</div>

---

LumiRealm은 [RisuAI](https://github.com/kwaroran/Risuai) 캐릭터 카드, 모듈, 로어북을 [Lumiverse](https://github.com/prolix-oc/Lumiverse) 안에서 네이티브로 실행하는 Lumiverse 확장입니다. RisuRealm 봇 브라우저가 내장되어 있습니다.

자세한 안내는 **[위키](https://github.com/AMousePad/LumiRealm/wiki)** 를 참고하세요.

## 스크린샷

|                  예시 카드                   |                       RisuRealm 탐색                       |
| :------------------------------------------: | :--------------------------------------------------------: |
| ![1778064388761](../image/README/1778064388761.png) | ![1778064256839](../image/README/1778064256839.png) |

|                      뷰어                      |                      상태                      |
| :--------------------------------------------: | :--------------------------------------------: |
| ![1778064299483](../image/README/1778064299483.png) | ![1778064443131](../image/README/1778064443131.png) |

## 설치

LumiRealm은 Lumiverse 확장으로 설치됩니다. Lumiverse 버전 **1.0.0 이상** 이 필요합니다.

1. Lumiverse 인스턴스를 엽니다.
2. **사이드바 → 아래로 스크롤 → 확장 탭**으로 이동하여 다음을 추가합니다:

   ```txt
   https://github.com/AMousePad/LumiRealm
   ```
3. 먼저 모든 권한을 허용하세요. LumiRealm은 모든 권한이 있어야 동작합니다. [이유는?](https://github.com/AMousePad/LumiRealm/wiki/Architecture)
4. 확장을 활성화하면 사이드바에 **LumiRealm** 탭이 표시됩니다.

## 커뮤니티

- **[Discord](https://github.com/AMousePad/LumiRealm/wiki/Discord)**: Lumiverse 서버에서 만나요!
- **[이슈](https://github.com/AMousePad/LumiRealm/issues)**: 버그 신고와 기능 요청.
- **[위키](https://github.com/AMousePad/LumiRealm/wiki)**: 사용자 가이드와 아키텍처 심층 분석.

## 라이선스

**GPL-3.0-or-later.** LumiRealm은 [RisuAI](https://github.com/kwaroran/Risuai) (GPL-3.0, © 2024 Kwaroran)의 파생 저작물입니다. 전체 모듈(Lua 브리지 prelude, 모든 CBS 핸들러, `processScriptFull` 포팅, 모듈/토글 DSL 파서, 로어북 데코레이터 파서)은 직접 포팅된 것입니다. Risu의 컴파일된 CSS 번들은 그대로 포함됩니다.

<p align="right">(<a href="#readme-top">맨 위로</a>)</p>
