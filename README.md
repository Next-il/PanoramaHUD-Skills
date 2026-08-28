# CS2 Panorama Skills

Agent skills for building CS2 Panorama HUD menus - the kind driven by the `custom_hud_layout` entity
that shipped in the 2026-08-24 update.


|                                         |                                                                                                                |
| --------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| [`cs2-panorama-hud/`](cs2-panorama-hud) | Authoring layouts, driving them from a CounterStrikeSharp plugin, and previewing them without a VPK round trip |




## Why this exists

Panorama looks enough like the web that a model writes web CSS at it with total confidence. And
Panorama **drops what it does not recognise without a word** - no error, no warning, the rule simply
does nothing.

`display: flex`, `rgba()`, `calc()`, `::before`, `background-size: contain`, web-order `box-shadow`.
Every one of those is wrong here. Every one fails silently.

That is what makes this worth packaging. The usual skill teaches an unfamiliar API; this one exists
to **unlearn a familiar one**. If Panorama errored on bad CSS a model would self-correct on the next
iteration and the skill would be redundant. It does not, so it will not.

Everything in it was learned by hitting the wall. Some examples that each cost a full compile-pack-
copy-restart-join cycle to find:

- `background-size` takes `contains`, not `contain`. The wrong spelling falls back to `auto`, which
means the image's original size, which overflows.
- `box-shadow` takes the colour **first**.
- A child at `width: 100%` inside a parent with no width renders as **nothing** - the parent sizes to
its children, the child sizes to its parent, and the circle resolves to zero.
- There is no wrapping. Five-per-line means five panels per line, structurally.
- An `<Image src>` cannot be changed by the server, so a dynamic picture has to be a class setting
`background-image` on a `Panel`.



## What is in it

```
cs2-panorama-hud/
  SKILL.md                                what the model reads
  references/panorama-css-reference.txt   the full CSS vocabulary, 140 properties
  scripts/new_layout.py                   scaffold a layout + stylesheet + the C# to drive it
  scripts/validate.py                     gate: XML, tag/attribute whitelist, CSS names AND values
  scripts/preview.py                      render a layout to browser-viewable HTML
  scripts/build-hud.ps1 / .cmd            compile and deploy without opening Workshop Tools
```

The scripts are plain Python 3 with no dependencies and are **useful on their own**, agent or not.
`validate.py` exits non-zero, so it can gate a build.

## Confidence

The two halves are not equally solid, and the skill says so where it matters.

**The CSS vocabulary is authoritative.** `panorama-css-reference.txt` is read out of
`libpanorama.so`: 140 registered properties with Valve's own doc strings, the selectors, the
at-rules, the value grammars, and an explicit list of what is *not* there. Regenerate it after a game
update with the `dump_panorama_css_properties` console command.

**The XML whitelist is empirical.** `custom_hud_layout` validates against something much narrower
than Panorama's own rules and reports one violation per load, so it was mapped by hitting it. Treat
it as a lower bound, not a specification.

## Installing

A skill is a folder. There is no installer.


|                           |                                                            |
| ------------------------- | ---------------------------------------------------------- |
| Claude Code / Claude apps | copy `cs2-panorama-hud/` into your skills directory        |
| Cursor                    | put `SKILL.md`'s contents in `.cursor/rules/` as an `.mdc` |
| Anything else             | feed `SKILL.md` as project instructions                    |


The format is Anthropic's, but the content is markdown - the rules travel, only the wrapper changes.

## What it will not do

A skill is instructions, not enforcement. It makes a model much likelier to write correct Panorama;  
it does not stop it writing `display: flex` anyway. That is exactly why `validate.py` exists and why  
the skill tells the agent to run it before every compile.

