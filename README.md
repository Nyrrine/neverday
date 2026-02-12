# jellyfish!!! — study notes + credits

## Credit

This was a **study project**: I started from an idea I loved and rebuilt/extended it to learn more about p5.js, timing/parametric motion, and shader-style post effects.

The sketch is **inspired by** Yuruyurau’s original p5.js one-liner.

**Original post (Yuruyurau)**: [x.com post](https://x.com/yuruyurau/status/2005652330612736419)

Original code (as shared):

```js
t=0,draw=$=>{t||createCanvas(w=400,w);background(9).stroke(w,46);for(t+=PI/30,i=3e4;i--;)d=mag(k=i<2e4?sin(i/9)*9:4*cos(i/49)*cos(i/3690),e=i/984-12)**2/99+1,point((q=k*(4+sin(d*16-t+k))-5*sin(atan2(k,e)*9))+60*sin(c=d*1.1-t/18+i%2*3)+200,(q+40)*sin(c-d)+d*79)}
```

## What I did differently

The original is a compact 2D p5 sketch that draws ~30k points per frame with a single stroke style.

My version keeps the same “parametric points from trig + distance + angle” core, but changes how it’s rendered and finished.

I don’t draw directly to the main canvas. Instead I render into an **offscreen buffer** (`createGraphics`) by stamping 1×1 pixels (`fillRect(x, y, 1, 1)`), then draw that buffer onto the main canvas.

I also use a **WEBGL** main canvas so I can run post-processing passes as full-screen filters.

From there I add two post effects:

- Bloom/glow: a small weighted blur mixed back into the image.
- Ordered dithering: a 4×4 Bayer matrix to gently quantize color and add texture.

Color-wise, I alternate between two translucent fills to get a layered look before the filters run!!! The colors are actually based off of march seventh and evernight from hsr!

## The math (what’s going on)

Both versions build a moving “field” of points from a loop index `i` and time `t`, using common trig + polar concepts.

### The ingredients

- **Trigonometric oscillators**: `sin(...)`, `cos(...)`
  - Provide periodic motion and interference patterns.
- **Distance (radius) from the origin**: `mag(k, e)`
  - In p5.js, `mag(k, e) = sqrt(k² + e²)`.
- **Angle (polar coordinate)**: `atan2(k, e)`
  - Gives an angle from the origin; using it inside `sin(...)` introduces rotational symmetry / angular banding.
- **Nonlinear warps**: nested trig and multiplication
  - Small changes in `d` and `atan2(...)` create large spatial “folds,” which is why the pattern looks organic.

### A readable breakdown of the original formulas

The code defines two intermediate coordinates, `k` and `e`, from the loop index `i`:

- `k` switches behavior halfway through the loop:
  - For `i < 20000`: `k = sin(i/9) * 9`
  - Else: `k = 4 * cos(i/49) * cos(i/3690)`
- `e = i/984 - 12` (a slow linear ramp as `i` changes)

Then it computes:

- `d = mag(k, e)^2 / 99 + 1`

Since `mag(k,e)^2 = k^2 + e^2`, you can think of this as:

- `d ≈ (k^2 + e^2)/99 + 1`

So `d` is basically a **scaled radius-squared** term. That’s important because it grows with distance and becomes a strong driver for phase.

Next:

- `q = k * (4 + sin(d*16 - t + k)) - 5 * sin(atan2(k,e) * 9)`

This is a **radial + angular warp**:
- `k * (4 + sin(...))` is an amplitude-modulated “push” along `k`.
- `sin(atan2(...) * 9)` introduces **9-fold angular structure** (because of `* 9`).

Then:

- `c = d*1.1 - t/18 + (i % 2) * 3`

This is a phase accumulator:
- `d*1.1` ties phase to radius,
- `-t/18` animates it over time,
- `(i%2)*3` offsets alternating points into two interleaved sets.

Finally, the point is plotted:

- `x = q + 60*sin(c) + 200`
- `y = (q+40)*sin(c-d) + d*79`

This mixes:
- a horizontal oscillation (`60*sin(c)`) and a centering offset (`+200`),
- a vertical warp that depends on both `q` and the phase difference `(c - d)`,
- plus a strong radius-dependent term (`d*79`) that “fans” the structure.

## Attribution note

This project is an **inspired-by study** since i mainly wanted to focus on exploring parametric trigonometric fields, polar coordinate warps (`mag`, `atan2`), and a modern “shader post-processing” pipeline on top of a point-plot sketch. mostly because everyone and their mama is trying dithering and i wanted to try bloom LOL

If you’re yuruyurau and want the credit wording adjusted or the repo taken down, please reach out and I’ll respond quickly, im a huge fan of ur work!!! <3333

