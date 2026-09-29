<a name="readme-top"></a>

<div align="center">

<img src="../logo.png" alt="LumiRealm" width="640"/>

[English](../README.md) | [한국어](readme-ko_kr.md) | [日本語](readme-ja_jp.md) | [简体中文](readme-zh_cn.md) | [繁體中文](readme-zh_tw.md) | [Deutsch](readme-de_de.md) | **Русский**

[![License](https://img.shields.io/badge/license-GPL--3.0--or--later-blue)](../LICENSE)
[![Lumiverse](https://img.shields.io/badge/Lumiverse-1.0.0%2B-blueviolet)](https://github.com/prolix-oc/Lumiverse)
[![RisuAI](https://img.shields.io/badge/RisuAI-port-9cf?logo=svelte)](https://github.com/kwaroran/Risuai)
[![TypeScript](https://img.shields.io/badge/TypeScript-5-blue?logo=typescript)](https://www.typescriptlang.org/)
[![Bun](https://img.shields.io/badge/Bun-bundle-fbf0df?logo=bun)](https://bun.sh)

</div>

---

LumiRealm — это расширение [Lumiverse](https://github.com/prolix-oc/Lumiverse), которое запускает карточки персонажей, модули и лорбуки [RisuAI](https://github.com/kwaroran/Risuai) непосредственно внутри Lumiverse. Включает встроенный браузер ботов RisuRealm.

Полное руководство — в **[Wiki](https://github.com/AMousePad/LumiRealm/wiki)**.

## Скриншоты

|              Пример карточки               |                       Поиск в RisuRealm                       |
| :----------------------------------------: | :----------------------------------------------------------: |
| ![1778064388761](../image/README/1778064388761.png) | ![1778064256839](../image/README/1778064256839.png) |

|                    Просмотрщик                    |                  Состояние                  |
| :--------------------------------------------: | :--------------------------------------------: |
| ![1778064299483](../image/README/1778064299483.png) | ![1778064443131](../image/README/1778064443131.png) |

## Установка

LumiRealm устанавливается как расширение Lumiverse. Требуется Lumiverse версии **1.0.0 или новее**.

1. Откройте свой инстанс Lumiverse.
2. Перейдите в **Боковую панель → Прокрутите вниз → Вкладка расширений** и добавьте:

   ```txt
   https://github.com/AMousePad/LumiRealm
   ```
3. Сначала включите все разрешения. LumiRealm нужны все они для работы. [Почему?](https://github.com/AMousePad/LumiRealm/wiki/Architecture)
4. Включите расширение. Вкладка **LumiRealm** появится в боковой панели.

## Сообщество

- **[Discord](https://github.com/AMousePad/LumiRealm/wiki/Discord)**: На сервере Lumiverse!
- **[Issues](https://github.com/AMousePad/LumiRealm/issues)**: отчёты об ошибках и пожелания.
- **[Wiki](https://github.com/AMousePad/LumiRealm/wiki)**: руководство пользователя и подробный разбор архитектуры.

## Лицензия

**GPL-3.0-or-later.** LumiRealm — производная работа от [RisuAI](https://github.com/kwaroran/Risuai) (GPL-3.0, © 2024 Kwaroran). Целые модули (prelude Lua-моста, все обработчики CBS, перенос `processScriptFull`, парсер DSL модулей и переключателей, парсер декораторов лорбука) являются прямым переносом. Скомпилированный CSS-бандл Risu поставляется без изменений.

<p align="right">(<a href="#readme-top">наверх</a>)</p>
