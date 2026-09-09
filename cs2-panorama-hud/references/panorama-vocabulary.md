# Panorama vocabulary, and what a custom_hud_layout can reach

Companion to `panorama-css-reference.txt`, which is the raw per-property dump. This file carries the
things that dump only implies: the selector and attribute vocabularies, and - for each - whether a
**server-driven custom_hud_layout** can actually use it or whether it is full-Panorama-only.

Three labels are used throughout, and nothing is left ambiguous:

- **Usable** - works in a custom_hud_layout, evidenced by the kit or by this project.
- **Testable** - registered, nothing about the custom_hud_layout sandbox blocks it, but nobody here
  has run it. Try it before you design around it.
- **Full Panorama only** - Valve's client UI uses it; a custom_hud_layout cannot. Either the tag or
  attribute is off the whitelist, or it needs scripting.

Valve's own layouts are evidence of what the **engine renders**, never of what will **load**. See
`valve-layout-corpus.md` for how narrow the overlap actually is (8 of Valve's 294 layouts).

---

## Panel states, as pseudo-classes

Registered vocabulary, all 13. The third column is the one that saves time.

| State | Needs | In a custom_hud_layout |
|---|---|---|
| `:hover` | mouse over the panel | **Usable.** Needs the menu's input capture, which PanoramaManager turns on |
| `:active` | mouse held down on the panel | **Usable**, same condition |
| `:hscroll` / `:vscroll` | the panel is scrolling | **Testable** - follows from `overflow: scroll`, unverified |
| `:selected` | the `selected` XML attribute, or script | **Full Panorama only** - `selected` is not whitelisted |
| `:disabled` / `:enabled` / `:activationdisabled` / `:parentdisabled` | the `disabled` XML attribute, or script | **Full Panorama only** - `disabled` is not whitelisted |
| `:focus` / `:descendantfocus` | `acceptsfocus`, or script | **Full Panorama only** - `acceptsfocus` is not whitelisted |
| `:inspect` / `:layoutfailed` | the Panorama debugger | Debug only |

So: two of thirteen. `:selected` in particular reads like the obvious way to highlight a chosen row -
it is not available. Toggle a class from C# instead, which is what the server can do anyway.

## Structural selectors

`:first-child`  `:last-child`  `:not( ... )`  `:nth-child( ... )`  `:nth-last-child( ... )`

All five are registered, none need scripting or an attribute, so all five are **Testable** and there
is no reason to expect any of them to fail. `:not()` and `:nth-last-child()` were previously recorded
in this skill as not existing. That was wrong.

Worth caring about because **C# class writes are the scarce resource** - they cost a netvar diff each
and panel ids, class names and dialog variable names share a 1024-per-entity intern cap
(`runtime-behaviour.md`). Anything the stylesheet can decide structurally is a write you do not send:

```css
.row:nth-child( 2n )          { background-color: #ffffff08; }   /* zebra, zero classes */
.row:not( .selected ) .badge  { opacity: 0.4; }                  /* one class drives the rest */
.row:last-child               { border-bottom: none; }           /* separators without a class */
```

No `::before` / `::after` - absent from every vocabulary in both extractions of the binary. No
`:nth-of-type`, no attribute selectors.

## At-rules

`@define`  `@import`  `@keyframes`. Anything else logs "Found unsupported CSS at-rule".

`@import` is how stock stylesheets come in (`s2r://.../csgostyles.vcss_c`) and it is the only one of
the three whose use is proven here.

`@define` is Panorama's named-value mechanism. `calc()` and `var()` truly do not exist, but "Panorama
has no variables" is false - it has this one. **Neither dump carries its syntax**, and no example
exists in this repo or in `references/kit/`, so the spelling is unverified. Other Source 2 Panorama
dialects are documented as `@define name: value;` referenced as `#name`; treat that as a hypothesis
worth one throwaway rule, not as fact. Write the answer here once someone runs it.

## Blend modes

`-s2-mix-blend-mode` takes: `normal` `multiply` `screen` `additive` `opaque` `overlay` `hardlight`
`linearburn` `darken` `lighten` `colordodge` `colorburn` `hue`.

`normal` is alpha blending, unlike web. An older extraction listed `softlight` and missed
`linearburn` and `opaque` - the 13 above come from the property's own doc string, so prefer them;
`softlight` is unverified.

## Transforms

Registered: `rotate3d` `rotatex` `rotatey` `rotatez` `scale3d` `scalex` `scaley` `scalez` `skew`
`skewx` `skewy` `translate3d` `translatex` `translatey` `translatez`.

The `transform` doc string names a **smaller** set - translate3d/x/y/z, scale3d, rotate3d/x/y/z - and
omits every skew and scalex/y/z. An older extraction listed bare `rotate`, `scale` and `translate`,
which appear in neither. The doc-string set is the safe subset; everything else is unverified.

## Layout file structure

A .vxml may contain: `id` `root` `script` `scripts` `snippets` `styles`.

In a custom_hud_layout only `styles`, and `include` inside it, survive. `scripts`, `script` and
`snippets` are rejected. That is not a corner case: 189 of Valve's 294 layouts declare scripts and
103 declare snippets, so most Valve layouts cannot be a starting point at all.

## Panel XML attributes

Full Panorama registers 58. A custom_hud_layout accepts **seven**:

```
id  class  hittest  text  src  texturewidth  textureheight
```

The other 51 are full-Panorama-only and any one of them is a hard load rejection
("Layout contains disallowed attribute X for panel type 'Y'"). The ones you will actually reach for,
because Valve's layouts are full of them:

| Attribute | Uses in Valve's 294 layouts | What to do instead |
|---|---|---|
| `onactivate` | 137 (49 of them on `<Button>`) | Nothing in the layout. Clicks arrive through PanoramaManager's channel keyed on the panel **id** |
| `html` | 127, almost all on `<Label>` | Plain `text`. No markup channel exists |
| `useglobalcontext` | 107 | Not applicable - a custom_hud_layout has no script context to share |
| `scaling` | 94, all on `<Image>` | No fit mode is available on `<Image>`. Use a `Panel` with `background-image` + `background-size` |
| `onload` | 63 | No load hook. Author the state you want and toggle from C# |
| `style` | 59 | A class. `style` is rejected even though it is valid Panorama |
| `hittestchildren` | 32 | Only `hittest` is available |
| `acceptsfocus` / `acceptsinput` | 25 / 24 | Not available, which is also why `:focus` is unreachable |
| `selected` / `disabled` | 4 / 7 | Not available, which is why `:selected` and `:disabled` are unreachable |
| `tooltip` / `placeholder` / `snippet` / `tabindex` / `value` / `defaultsrc` | 17 / 15 / 17 / 12 / 12 / 10 | None available |

Full list of the 58 is in `panorama-css-reference.txt` under PANEL XML ATTRIBUTES.

---

## Recipes the raw dump only implies

### Fit text to a box instead of counting characters

```css
.cell { width: 220px; text-overflow: shrink min( 10px ) ellipsis; }
```

`text-overflow` takes `clip` | `ellipsis` (**the default**, contrary to web) | `shrink` | `noclip`.
`shrink` lowers the **font size** until the text fits. `shrink min( 10px )` stops shrinking at 10px
and clips the rest; `shrink min( 10px ) ellipsis` stops at 10px and ellipsises the rest.

This is the answer to hand-computing pixel budgets against the longest expected string. It is also
strictly better than letting `overflow: squish` fire, because a squish scales the **whole row** -
icons and glyphs along with the text - while `shrink` only touches one label's font.

### Bars, gauges and dials without `width`

```css
.bar { width: 100%; transition-property: clip; transition-duration: 0.4s; }
.bar.w10 { clip: rect( 0%, 50%, 100%, 0% ); }        /* top, right, bottom, left */
.dial.q1 { clip: radial( 50% 50%, 0deg, 90deg ); }   /* centre, start angle, swept angle */
```

Valve: *"This clipping has no impact on layout, and is fast and supported for
transitions/animations."* That sentence is the whole reason to use it - a `width` transition restarts
when any dialog variable in the same subtree is written (`runtime-behaviour.md`), and `clip` does not.

**Honest limit:** `clip` does not shrink the class ladder. The server can only toggle classes, so an
N-step bar is still N classes either way. What `clip` buys is (a) immunity to the text-update
re-layout, and (b) with `radial`, a shape `width` cannot express at all - cooldown dials, radial
sweeps, pie timers.

Valve's doc string types the radial example as `radial( 50% %50, 0deg, 90deg )`. `50% 50%` is the
obvious reading of the typo, but it is unverified.

### Scale a whole HUD without re-deriving every pixel

```css
.Root.ui-large { ui-scale: 150%; }
```

*"This scale happens at the layout level rather than the bitmap level, so things like text will
increase their font size rather than just bitmap scaling."* Takes one value or three
(`ui-scale: 50% 100% 150%` for X, Y, Z).

This is one of the very few genuinely global knobs a server can drive, because it works from a class
on the root panel - which is exactly the one thing C# can toggle. A per-player HUD-scale option is
three classes, not a second stylesheet.

### The three blurs are three different targets

| Property | Blurs | Cost |
|---|---|---|
| `blur` | this panel and all its children | composition pass |
| `background-blur` | whatever is **behind** this panel, at composition | composition pass. Recorded as not working in `runtime-behaviour.md`, and Valve's own stylesheets use it zero times |
| `world-blur` | the world / backbuffer **before** drawing | cheapest route to heavy blur, via mipmapgaussian |

All three take `gaussian( hstd, vstd, passes )`, or `gaussian( n )` as shorthand for both directions
and one pass. Good standard deviations are 0-10, and **more than one pass is bad for perf** - Valve
says so explicitly. If 10 is not enough, `world-blur` alone also takes
`mipmapgaussian( 6, 6, 4 )`, where each pass is preceded by a quarter-area downsample.

They are not substitutes. Reaching for `world-blur` because `background-blur` did not work only gives
the right picture when the thing you wanted blurred was the world.

### Composition-time colour, all class-toggleable, all inherited by children

`wash-color` (tint, alpha = intensity) · `saturation` (1.0 none, 0.0 grey, >1 over) · `brightness`
(HSB multiplier) · `contrast` · `hue-rotation` (degrees) · `opacity` · `opacity-brush` (takes a
gradient) · `-s2-mix-blend-mode`.

Every one of these applies to the panel **and all its children**, which makes one class enough for a
whole "disabled" or "danger" or "selected" look - instead of duplicating a palette across every child
rule. Only `wash-color` is used in this repo today.

```css
.row.spent { saturation: 0.2; brightness: 0.7; }   /* whole row, one class */
```

### box-shadow has shape keywords, and a negative blur

Order is `[inset|fill|hollow] color hoff voff blur spread`. `inset` is an inner shadow or glow,
`fill` sits behind the whole box, `hollow` is clipped to outside the border area only. A **negative**
blur radius gives a hard-edged look - "effectively a rounded outline of the same size as the blur",
which is a rounded outline without spending the `border` property on it.

### background-position takes four values

`<left|center|right> <h length> <top|center|bottom> <v length>`. A **percent** anchors that point of
the image over the same point of the panel; **pixels** offset the image's top-left from the named
alignment keywords. One value means the other is `center`.

```css
background-position: left 10px top 40px;   /* 10px right of, 40px below the panel's top-left */
```

### Second tier, one line each

- `border-radius` per corner, with a horizontal/vertical split:
  `border-radius: 2px 3px 4px 2px / 2px 3px 3px 2px` (top-left, top-right, bottom-right, bottom-left).
  `border-radius: 50% / 50%` is a circle.
- `border-image` - nine-slice, e.g. `border-image: url( "file://msg.png" ) 25% / 1 / 20px repeat`.
- `text-shadow: 2px 2px 8px 3.0 #333333b0` and `img-shadow: 2px 2px 8px 3.0 #333333b0 alpha-only`.
  Both carry a **strength** term between blur and colour, which is not a web thing; `img-shadow` also
  takes `alpha-only | legacy | point` and is only meaningful on images.
- `line-height: normal | 20px | 1.2 | 120%` - Valve notes there is no web-style inheritance weirdness.
- `letter-spacing: normal | 1px`, `paragraph-spacing`, `text-align: justify | justify-letter-spacing`
  - all four are only meaningful once text actually wraps, so pair them with `white-space: normal`.
- `transition-frame-time: 0.2s` - a deliberate low-framerate look. `transition-high-framerate: true`
  asks for the opposite.
- `texture-sampling: normal | alpha-only | point` - `point` for pixel art, `alpha-only` to drive all
  three colour channels from the texture's alpha.
- `font-stretch: normal | condensed | expanded`.
- `text-decoration-style: none | dashed | dotted | wavy`.
- `layout-position: static | fixed` - `fixed` positions normally but ignores the parent's scroll offset.
- `border-brush` - a gradient across the whole border paint area. Valve marks it **EXPERIMENTAL**.
- `cubic-bezier( ... )` is accepted by `transition-timing-function`, though not by the `transition`
  shorthand's documented list (ease, ease-in, ease-out, ease-in-out, linear).
