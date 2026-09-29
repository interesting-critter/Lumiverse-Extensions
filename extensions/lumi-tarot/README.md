<!-- Mirror of upstream documentation. Everything below the metadata table is the original file, unmodified. -->

| Field | Value |
| --- | --- |
| Extension | LumiTarot |
| Source repository | https://github.com/interesting-critter/lumi-tarot |
| Original link (from Lumiverse-Extensions README) | https://github.com/interesting-critter/lumi-tarot |
| Upstream path | `README.md` |
| Retrieved | 2026-09-29 @ `main` (`6e926d7`) |

---

# lumi-tarot
Just a simple, mostly vibecoded tarot reading extension.

* Any character can perform the reading on {{user}}. Either let the current chat's character do it or choose one from your library.
* The main prompt can be edited in the settings tab.
* The reader receives your persona description along with the current chat's character description and limited chat history.
* Includes 6 different spreads:
  * Single Card
  * Past/Present/Future
  * Mind/Body/Soul
  * Five Card Cross
  * Seven Card Horseshoe
  * Celtic Cross (Ten Card)
* Uses your chosen LLM connection to interpret each card as it's flipped, then a "synthesis" of all cards together at the end.
* The three most recent readings per chat are saved between sessions in the history tab. History can be cleared for current chat or all chats in the settings tab.
