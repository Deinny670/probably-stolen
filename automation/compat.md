# Automation (`danny.automation`): how it recognises your items

> 给别的 mod 作者看的说明（D39 方案 A，Danny 2026-11-22 定：现在写说明，发布后有人要再开 API）。发布时放进 release 文件夹 / 工坊说明，所以用英文写。内容必须和代码一致，改了相关代码要同步改这里。

This page is for mod authors who want their items to work with the Executor. There is no API yet. Almost everything is read from the game's own data, so an item built the way vanilla items are built usually works without any extra work. If you need something this page does not cover, ask; a small registration API is planned after release.

## Subjects (what a logic works on)

- The player clicks any owned item to add it as a subject. Nothing is filtered by ID.
- Shift+click adds "every item like this in its container": the container is the item's parent (a cage, a machine, a box), or the storage area it sits in. Items are grouped by:
  - species for animals (`ANIMAL_ITEM_TAG`);
  - otherwise the custom name, if the item has one;
  - otherwise `identifier`.
- A subject is found again by `GetUniqueID()`, so items must keep their unique ID across saves (vanilla items do).

## Attributes (what an "If" can test)

An attribute is offered when its rule matches the item. All values are read from `item.state` tags.

| Your item has | It gets |
| --- | --- |
| `LIQUID_CONTAINER_TAG` + `LIQUID_CONTAINER_CAPACITY` + the usual liquid part tags | Liquid (mL), Fill %, Empty, Full, Temperature (`CONTAINER_TEMP_TAG`); with an Aquascan or Chem Scanner: water purity, impurities, every liquid in `Liquid.Liquids` (mL and %) |
| A content window that is not a liquid container | Items inside, Animals inside |
| Either of the above | Seconds unchanged |
| `power_source_item` + `power_source_item_energy` / `power_source_item_max_energy` | Charge % |
| `CAGE_HEALTH_TAG` / `CAGE_MAX_HEALTH_TAG` | Cage condition % |
| `INGOT_PURITY_TAG` (vanilla tier names) | Ingot purity tier (needs a metal scanner) |
| `WINE_CONTAINER_TAG` | Fermenting, Wine finished |

- **New liquids**: register them in the game's `Liquid.Liquids` the normal way. They show up in the scanner attribute list by themselves.
- **New numbers of your own** (for example "pressure" on your machine): these cannot be added yet. This is what the planned API is for.

## Buttons (Button Finger)

- The Button Finger presses buttons in the target's own window. It walks `contentWindow` → `MachineryPanelRow` → `MachineryPanel` and looks inside `MachineryStack`, `MachineryIntake` and `GridPixelElement`, collecting every `MachineryButton`.
- The label shown to the player is the button's text. If the button has no text, its `description` is used.
- **For your machine to work**: build its buttons as `MachineryButton` inside the usual machinery window, and give each one text or a description. A button the finger picked is found again by label first, then by position. Keep labels stable between versions.

## Tools

- The tool slot takes a fixed list: the siphon, the button finger, the labeler (writes a label value on targets), the wire cutter, the surgery tool, knives, the marking pen, the portable water purifier and the UV filter.
- Other items go into the supplies grid; the executor uses them on or puts them into a target the way a player would.
- The blue labeler, the magnifier, the spectral analyser, the dossier and keycards are refused (neither slot takes them).
- A modded tool cannot join the tool slot list yet (planned API).

## Things we patch (so you can avoid clashes)

- `HopperHelper`: `RunWindow`, `RunUnguarded`, `Passes`, `OpenWindow`, `OpenOutletWindow`, `Link`, `CanLinkPair`, `FindIntakesOf`, `IsOnAdd`, `IsOnMove`, `SetColor`, `CreateTooltip` (smart hoppers only; vanilla hoppers behave as before)
- `StorageExpansionHelper.RestoreTabs`
- `SaveManager.DecodeNodes`
- `ItemPinHelper.BlocksClicks` (getter, only while our picker is picking)
- `GameItemElement.ResolveSpriteByName` (only for our own sprites)
- `CustomUIBuilder.AddButton`, `AddFlexibleSpace`, `End`
- `StoreClientList.PlaceInventorInventory`
- `StoreClientListStory.CreateInventorModuleIntro` (adds to the end action of its first line)
- The bug-report client adds postfixes to `StoreClientManager.AddClient` and `PlayerStore.GetNextClient` (it only logs)

Vanilla items are never changed. Our own items use the `auto_` ID prefix.
