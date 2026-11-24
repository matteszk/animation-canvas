# AnimationCanvas

A tiny, dependency-free helper for `<canvas>` animations. You write the drawing; it handles the parts every canvas animation needs and nobody wants to rewrite.

**[▶ See the demos](https://matteszk.github.io/animation-canvas/demo/)**

- **Sizing.** The canvas fills its container, redraws when that changes size, and stays sharp on high-DPI screens. You draw in CSS pixels.
- **The frame loop,** with `time` (elapsed) and `delta` (since the last frame), so animations run at the same speed on 60 Hz and 120 Hz screens.
- **Pausing for free.** Nothing runs while the canvas is scrolled off-screen or the tab is hidden.
- **Reduced motion.** When the visitor prefers reduced motion, it draws one still frame instead of animating.
- **Image preloading** and **pointer tracking** for interactive pieces.

About 1.6 KB minified and gzipped. One ES module, no build step.

## Quick start

```html
<canvas id="hero" style="width: 100%; height: 400px"></canvas>

<script type="module">
  import { AnimationCanvas } from './src/animation-canvas.js';

  new AnimationCanvas('#hero', {
    draw({ ctx, width, height, time }) {
      ctx.clearRect(0, 0, width, height);
      const x = width / 2 + Math.sin(time / 600) * (width / 3);
      ctx.fillStyle = '#e7b84a';
      ctx.beginPath();
      ctx.arc(x, height / 2, 20, 0, Math.PI * 2);
      ctx.fill();
    },
  });
</script>
```

Size the canvas with CSS. AnimationCanvas sets its pixel size to match.

## Options

```js
new AnimationCanvas(canvasOrSelector, {
  draw(frame) {},         // required: called every frame
  setup(frame) {},        // once, after images load, before the first frame
  resize(frame) {},       // after the canvas changes size, e.g. to rebuild a grid
  images: ['a.png'],      // preload these; they arrive in frame.images
  pointer: false,         // track the pointer in frame.pointer
  autoplay: true,         // start playing as soon as it's visible
  animate: 'auto',        // 'auto' follows prefers-reduced-motion; true or false to force
  maxPixelRatio: 2,       // cap devicePixelRatio to limit work on very dense screens
  maxDelta: 100,          // cap delta (ms) so a long pause doesn't make things jump
});
```

Every callback receives the same `frame` object:

| Property | |
|---|---|
| `ctx` | The 2D context, already scaled for the pixel ratio |
| `width`, `height` | Size in CSS pixels |
| `time` | Milliseconds spent playing (pauses don't count) |
| `delta` | Milliseconds since the previous frame |
| `pointer` | `{ x, y, inside }` in CSS pixels, when `pointer: true` |
| `images` | Preloaded images, in the order given |
| `animated` | `false` when drawing a still frame for reduced motion |
| `anim` | The instance |

## Methods

| Method | |
|---|---|
| `play()` | Start or resume |
| `pause()` | Pause; the current frame stays on screen |
| `render()` | Draw one frame now, playing or not |
| `destroy()` | Stop and remove every listener and observer |

`anim.playing` tells you whether it's set to play. It may still be idle while off-screen.

## Tips

- **Use `delta` for movement** (`x += speed * delta`) so speed doesn't depend on the screen's refresh rate.
- **Handle the still frame.** When `frame.animated` is `false`, draw a pleasing static composition rather than frame zero.
- **Rebuild layouts in `resize`.** Grids and particle fields usually depend on the canvas size.

## Background

I first wrote a version of this for the animated sections of a large corporate marketing site, where a page could have half a dozen canvases that all needed the same setup. This is a from-scratch rewrite as a standalone library.

## License

MIT
