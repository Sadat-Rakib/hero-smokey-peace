# Smokey Peace Header

A full-viewport hero header for the luxury real-estate brand "FANCE" featuring a dark autoplaying video background overlaid with a real-time WebGL fluid-simulation ink smoke effect that responds to cursor movement, painting glowing teal/aqua wisps of smoke wherever the mouse moves.

## Tech stack
- React (client component, uses hooks: useRef, useEffect).
- Raw WebGL (via canvas.getContext('webgl')). No three.js or external WebGL library — the fluid solver is hand-written GLSL. No framer-motion, no icon library.

## Fonts & global styles
- Import via: `@import url('https://fonts.googleapis.com/css2?family=Dancing+Script:wght@700&family=Parkinsans:wght@300;400&display=swap');`
- Dancing Script (700) is used ONLY for the logo wordmark.
- Parkinsans (300, 400) is used for nav links, heading, and CTA text.
- Keep all styles scoped to this section (prefixed classes). Do not set any global body/html styles.

## Section container
- `position: relative; width: 100vw; height: 100vh; background: #FFFFFF; overflow: hidden;`
- Stacking order (z-index): canvas wrap = 0; navbar and centered content = 10.

## Structure, section by section
1. **Hidden video source** — a `<video>` element with `display: none`, `autoPlay loop muted playsInline`, `crossOrigin="anonymous"`, src `/proxy-video/Header%20Section/Smokey%20Header/Modern_house_with_infinity_pool_202609031459.mp4`. This video is never shown directly — it is sampled into a WebGL texture each frame and used as the base image behind the smoke.
2. **Canvas layer** — a div `.smokeH-canvas-wrap` (`position:absolute; inset:0; z-index:0`) containing a full-size `<canvas>` (`width:100%; height:100%; display:block`). The WebGL fluid simulation renders here.
3. **Top-left logo** — `.smokeH-navbar-left` positioned `absolute; left:3.75%; top:2.53%`, `display:flex; align-items:center; gap:6.09px`. Contains: an SVG mark (`.smokeH-vector`, 41.44px x 28.42px box; inner svg 36.24px x 22.74px, `fill:#FFFFFF`, viewBox "0 0 36 23", path `M0 6.64258L36.24 0.892578V5.07258L6.48 9.53258L30.96 8.18258L29.64 12.0526L8.4 12.6826L20.88 15.6526L18 19.3426L0 6.64258Z`) followed by the wordmark "FANCE" in Dancing Script 700, font-size 25.7349px, line-height 31px, letter-spacing -0.02em, `font-variant: small-caps`, color #FFFFFF.
4. **Top-right nav** — `.smokeH-navbar-right` positioned `absolute; right:3.75%; top:20px`, `display:flex; gap:5px`. Four pill buttons with labels "Home", "About", "Journal", "Contact". Each button: `padding:5px 10px; height:30px; background: rgba(255,255,255,0.1); backdrop-filter: blur(2px); border-radius:8px; cursor:pointer; transition: background 0.3s ease`. Text is Parkinsans 400, 15px/20px, #FFFFFF.
5. **Centered content** — `.smokeH-content-center` positioned dead-center via `absolute; top:50%; left:50%; transform: translate(-50%,-50%)`, `display:flex; flex-direction:column; align-items:center; gap:36px; width:100%; z-index:10; pointer-events:none`. Contains:
   - Heading `<h1>` (`.smokeH-heading`): Parkinsans 300, font-size 72px, line-height 82px, text-align center, color #FFFFFF, max-width 800px, margin 0. Text: "A Quiet Escape," then a `<br/>` then "Beautifully Designed".
   - CTA button `.smokeH-cta` (`pointer-events:auto`): `padding:13px 24px; height:46px; background:#FFFFFF; border-radius:8px; overflow:hidden`. Label "Explore the Space" in Parkinsans 400, 18px/20px, color #19120E.

## Assets — every URL
- Video (sampled as WebGL texture, never shown): `/proxy-video/Header%20Section/Smokey%20Header/Modern_house_with_infinity_pool_202609031459.mp4`
- Google Fonts: `https://fonts.googleapis.com/css2?family=Dancing+Script:wght@700&family=Parkinsans:wght@300;400&display=swap`

## Animations
- **Nav & CTA hover roll-up**: each button contains its label duplicated in two stacked `<span>`s inside a fixed 20px-tall overflow-hidden column. On hover the spans translate up by 100% to reveal the second copy. Transition: `transform 0.35s cubic-bezier(0.68, -0.35, 0.27, 1.35)`. Nav buttons also change background to #000000 on hover; the CTA changes background to #000000 and its text color animates to #FFFFFF (`color 0.3s ease`).
- **WebGL fluid smoke (the signature effect)**: a GPU stable-fluids simulation runs at 256x256 into ping-pong float framebuffers. Three GLSL passes per frame: (1) velocity — diffuse + damp (0.965) + add a large soft mouse force where `smoothstep(0.20, 0.0, distanceToMouse)`; (2) dye advection — advect along velocity, gentle decay (0.9965), inject glowing color at the cursor with an fbm-noise softened gaussian falloff `pow(max(0.0, 1.0 - softDist/0.16), 2.5)`; (3) render — screen-blend the dye over the video base texture: `col = 1.0 - (1.0 - base) * (1.0 - dye * 1.6)`. Ink is only injected while the mouse is actually moving (velocity threshold ~0.005), so it never blobs when stationary. Inject color oscillates over time between a teal palette (COOL1 [0.25,0.85,0.9], COOL2 [0.6,0.95,1.0]) and soft seafoam (WARM1 [0.45,0.8,0.75]).

Core render blend (fragment shader):
```glsl
vec3 base = texture2D(uImage, vUv).rgb;
vec3 dye  = texture2D(uDye, vUv).rgb;
vec3 col = 1.0 - (1.0 - base) * (1.0 - dye * 1.6);
gl_FragColor = vec4(clamp(col, 0.0, 1.0), 1.0);
```
Dye injection falloff:
```glsl
float influence = pow(max(0.0, 1.0 - softDist / 0.16), 2.5) * uMouseActive;
dye += uInjectColor * influence * 0.9;
```
Use `OES_texture_float` when available (fallback UNSIGNED_BYTE). Canvas DPR is clamped to 2. Sample the video into the base texture each frame only when `video.readyState >= HAVE_CURRENT_DATA`, with `UNPACK_FLIP_Y_WEBGL` set true. Always clean up: cancelAnimationFrame, delete textures/framebuffers/programs, and remove resize + mouse listeners on unmount.

## Responsive behavior
- Base layout uses percentage positioning so it scales fluidly across widths.
- Breakpoint at max-width 768px: heading shrinks to font-size 40px / line-height 48px; nav button gap reduces to ~3px. The smoke/canvas fills the viewport at all sizes.

## Key design principles
- The video is a hidden texture source only — all visible imagery is the WebGL composite.
- Cursor-reactive glowing teal ink over a dark base is the hero moment; keep the screen-blend and noise-softened injection intact.
- White typography over a dark scene; logo in script small-caps, everything else in clean sans.
- Centered content is `pointer-events: none` except the CTA, so cursor movement paints smoke everywhere.

## Common mistakes to avoid
- Do not display the video element — it must stay `display:none` and be sampled into WebGL.
- Do not use template literals in the component code; keep GLSL as concatenated strings or array-joined strings.
- Do not forget WebGL cleanup on unmount (memory leaks and multiple RAF loops otherwise).
- Keep SIM_SIZE at 256 and DPR clamped at 2 for performance.
- Set `crossOrigin="anonymous"` on the video or the texture upload will taint the canvas.

## Page title
Smokey Peace Header — FANCE luxury retreat hero with cursor-reactive WebGL ink smoke.

## Integration (build-safety — do not skip)
- Add this section as a **new** component file with a unique name. Don't edit or overwrite any existing file except to add its import and render it.
- Render it **after** all existing sections; keep every previously built section exactly as-is — never replace or remove them.
- If no project exists, create a minimal React + Tailwind app; if one exists, use it as-is — don't re-scaffold or change the Tailwind/build config or version.
- Keep it self-contained: scope its fonts and any resets to this section; never set global `body`/`html`/`*` styles or a global font.
- Install only the libraries this section names.