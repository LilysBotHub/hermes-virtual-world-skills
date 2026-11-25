# Dynamic Textures (osSetDynamicTexture*)

Render generated or external images in-world on the prim containing the script.

## Catalog

| Function | Threat | Purpose |
|---|---|---|
| `osSetDynamicTextureData` | VeryLow | Render a generated texture from drawing instructions |
| `osSetDynamicTextureURL` | VeryLow | Show an external URL image |
| `osSetDynamicTextureURLBlend` / `osSetDynamicTextureURLBlendFace` | VeryLow | Blended external images |
| `osSetDynamicTextureDataBlend` / `osSetDynamicTextureDataBlendFace` | VeryLow | Blended generated textures |

## osSetDynamicTextureData signature

```
key osSetDynamicTextureData(string dynamicID, string contentType, string data, string extraParams, integer timer)
```

| Param | Meaning |
|---|---|
| `dynamicID` | UUID of a prior call — **NOT IMPLEMENTED** (ignored); pass `""` |
| `contentType` | `"vector"` → `data` is a **list** of drawing instructions |
| `data` | Drawing instructions (a list when contentType is `"vector"`) |
| `extraParams` | `width:256,height:256,alpha:0-255,bgcolor:Red,altdatadelim,setalpha,lossless:true|false` (comma-separated) |
| `timer` | **NOT IMPLEMENTED** (ignored); refresh via a `timer` event |

Returns the UUID of the newly created texture.

## extraParams reference

- `width` / `height` — texture size in pixels (e.g. `width:256`).
- `alpha` — alpha component, 0 = clear … 255 = solid.
- `bgcolor` — background color name (e.g. `bgcolor:Red`).
- `altdatadelim` — delimiter between draw commands.
- `setalpha` — integer alpha (alias of `alpha`).
- `lossless` — `true`/`false` (default `false`; added 0.9.1.1).

## Drawing commands

`data` is a list of string tokens. Supported commands:

| Command | Syntax |
|---|---|
| `fill` | `["fill", "Red"]` — set fill color |
| `rect` | `["rect", x, y, w, h]` or `["rect", x, y, w, h, "fillcolor"]` |
| `circle` | `["circle", cx, cy, r]` or with `"fillcolor` |
| `line` | `["line", x1, y1, x2, y2]` |
| `text` | `["text", x, y, "string", size, "color"]` |
| `clear` | `["clear"]` — clear the texture |

Example (draw a white label on a red board):

```lsl
default
{
    state_entry()
    {
        list draw = [
            "fill", "Red", "rect", 0, 0, 256, 256,
            "fill", "White", "text", 10, 120, "Hello Grid", 24, "White"
        ];
        osSetDynamicTextureData("", "vector", (string)draw, "width:256,height:256", 0);
        // NOTE: data must be joined as a delimited string; use altdatadelim if needed.
        llSetTimerEvent(2.0);   // refresh every 2 s
    }

    timer()
    {
        osSetDynamicTextureData("", "vector", (string)["fill", "Red", "rect", 0, 0, 256, 256], "width:256,height:256", 0);
    }
}
```

## Notes

- `dynamicID` and `timer` are placeholders today — use a `timer` event to re-call and refresh.
- `lossless:true` gives crisper text/shapes at a memory cost.
- To show a remote image without local generation, use `osSetDynamicTextureURL(url, face, blend)`.