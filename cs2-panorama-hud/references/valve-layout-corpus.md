# What Valve actually uses - 294 decompiled layouts

Tag, attribute and class frequencies across **294 decompiled Valve .vxml_c layouts** (CS2 client UI,
plus a handful of map and editor layouts). Counts are **number of layout files** the name appears in,
not instance counts.

**Read this as evidence of what the engine renders, never as evidence of what will load.** Valve's
client UI is full Panorama. A `custom_hud_layout` runs a far narrower validator. The numbers below
exist so you can tell "registered" from "actually used", and so you stop copying Valve markup that
cannot load here.

## The calibration number

**8 of the 294 layouts (2.7%) fall entirely inside the custom_hud_layout subset** - only the four
whitelisted tags, only the seven whitelisted attributes, no scripts, no snippets:

```
maps/editor/zoo/scripts/welcome          panorama/layout/base_world_panel
panorama/layout/btn_alert                panorama/layout/matchtiles/menu
panorama/layout/tooltips/stats/tooltip_playerstats_weaponsgraph
panorama/layout/tooltips/tooltip_model_panel_instructions_info
panorama/layout/tooltips/tooltip_model_panel_instructions_lights
panorama/layout/tooltips/tooltip_model_panel_instructions_scene
```

56 use only the four whitelisted tags. **189 (64%) declare scripts and 103 (35%) declare snippets**,
neither of which survives here. That is what "far narrower validator" means quantitatively: a Valve
layout is a bad starting point roughly 97 times in 100.

## Tags - 166 distinct, 4 legal

96 tags appear in 2 or more files, 70 appear once. Only the first four are on the whitelist.

| Legal here | Files |
|---|---|
| `Panel` | 280 |
| `Label` | 246 |
| `Image` | 173 |
| `Button` | 100 |

Everything below is **full Panorama only** and is a hard load failure in a custom_hud_layout:

```
TextButton 60        PopupCustomLayout 54 Frame 52             RadioButton 31
ItemImage 30         TextEntry 26         CSGOAvatarImage 25   DropDown 24
ToggleButton 22      TooltipPanel 20      ParticleScenePanel 15 CSGOBlurTarget 14
CSGOHonorIcon 12     ProgressBar 10       Movie 9              Carousel 9
InventoryItemList 8  Slider 7             CSGOSettingsSlider 7 CarouselNav 6
CCSGOSteamInputAction 6  MapPlayerPreviewPanel 5  UICanvas 5   SpiderGraph 5
CSGOSettingsEnumDropDown 5  MapItemPreviewPanel 4  TooltipContents 4  EconItemImage 3
CSGOMoneyPanel 3     CSGOCrosshair 3      JSDelayLoadList 3    ContextMenuManager 3
HTML 2               CountdownTimer 2     Countdown 2          ... ~130 more, nearly all CSGO*
```

`ProgressBar` and `Slider` are the ones people go looking for. They do not exist here - a bar is a
`Panel` with a `clip` class ladder.

## Attributes - 216 distinct, 7 legal

| | Files |
|---|---|
| **Whitelisted:** `class` | 1199 |
| `id` | 1069 |
| `text` | 308 |
| `src` | 200 |
| `hittest` | 185 |
| `textureheight` | 139 |
| `texturewidth` | 136 |

**Full Panorama only**, by frequency:

```
onactivate 137   html 127         useglobalcontext 107  scaling 94
onload 63        style 59         popupbackground 56    oncancel 38
onmouseover 33   onmouseout 33    group 33              hittestchildren 32
acceptsfocus 25  acceptsinput 24  menuclass 21          oninputsubmit 19
snippet 17       tooltip 17       placeholder 15        unlocalized 13
tabindex 12      value 12         defaultsrc 10         disabled 7   selected 4
```

## Per-tag, for the four tags that are legal

The useful cut: what a reader copying Valve will trip over.

| Tag | Whitelisted, and how often | Will be rejected |
|---|---|---|
| `Panel` | `class` 274, `id` 210, `hittest` 70 | `useglobalcontext` 66, `acceptsinput` 20, `acceptsfocus` 20, `style` 18, `hittestchildren` 16, `onload` 15 |
| `Label` | `text` 230, `class` 222, `id` 181, `hittest` 10 | **`html` 116**, `style` 12, `unlocalized` 11, `value` 10 |
| `Image` | `src` 149, `class` 147, `textureheight` 130, `texturewidth` 128, `id` 120 | **`scaling` 61**, `defaultsrc` 7, the five `svg*` attributes |
| `Button` | `class` 93, `id` 86, `hittest` 5 | **`onactivate` 49**, `onmouseout` 15, `onmouseover` 14 |

Two conclusions worth carrying into a design:

**`texturewidth`/`textureheight` on `<Image>` is Valve's default practice, not an optimisation.**
130 of 173 Image-using layouts set them. Meanwhile `scaling` - the fit mode, the thing that makes an
`<Image>` behave sensibly at a size you did not author - is **not** whitelisted. That is a concrete
second reason a `Panel` with `background-image` beats an `<Image>` here, on top of `src` being static.

**A `<Button>` copied from Valve is a dead control.** Half of them carry `onactivate`, which does not
exist in a custom_hud_layout. Clicks arrive through PanoramaManager's channel keyed on the panel id,
so the id is the wiring and the `<Button>` tag itself buys nothing a `Panel` with `hittest` does not.

## Valve ships misspelled attributes

`hitest` 2 · `hittext` 2 · `oncanel` 2 · `clas` 1 · `hitchildren` 1 · `texturehwidth` 1

In full Panorama an unknown attribute is silently ignored, which is why these shipped. In a
custom_hud_layout the identical typo is a hard rejection with a real message
("Layout contains disallowed attribute X for panel type 'Y'"). The validator here is stricter than
the engine Valve tests against - so the strictness is a feature, and it is also why you cannot
conclude anything about legality from the fact that a Valve layout loads.

## Stock utility classes worth importing

`runtime-behaviour.md` notes that stock stylesheets can be `@import`ed. These are the class names
those stylesheets actually define, recovered by frequency across the corpus - i.e. the ones Valve
itself relies on, so the least likely to be renamed:

```
layout      full-width 102        left-right-flow 101    top-bottom-flow 89     vertical-center 82
            horizontal-center 73  full-height 73         horizontal-align-right 63
            horizontal-align-left 30  vertical-align-bottom 28  right-left-flow 9
            text-align-center 8   left-right-padding 14  right-padding 10  right-margin 11
visibility  hidden 67             Hidden 53              hide 14
fonts       stratum-font 31       stratum-regular-condensed 28   stratum-regular 26
            stratum-medium-condensed 19  stratum-medium 18   stratum-bold 17
            stratum-medium-italic 14  stratum-bold-italic 10  stratum-bold-condensed 9
            stratum-bold-mono 8
sizes       fontSize-m 11         fontSize-l 10          fontSize-sm 9
colour      fontcolor-white 8     hud-colorize-wash 10   additive 10
```

**`hidden` and `Hidden` both exist**, in different stock sheets. If you `@import` csgostyles and then
name your own reveal class `hidden`, you have collided with a stock rule and the two will fight in
specificity order. Prefix your own classes.

## Dialog variables

115 of the 294 layouts use dialog variables - 280 distinct names, and the distribution is almost
flat (`title` 8, `name` 6, `item-name` 4, then a long tail of ones). Valve does not have a shared
naming convention to copy. Pick your own and keep it consistent with the C# side; `new_layout.py`
generating both halves is what actually prevents the mismatch.
