# React & Next.js Motion Lifecycle

Load for client boundaries, hydration, StrictMode, route transitions, and teardown. Most "the animation fires twice" and "it breaks after navigation" reports resolve here.

## Client boundaries

Animation touches the DOM, so it cannot run during server rendering.

- In the Next.js App Router, any component that animates needs `"use client"`.
- Keep the boundary tight: mark the small animated component, not the whole page, or interactivity cost spreads further than needed.
- Never import an animation library into a server component. Even unused, it pulls into the wrong graph.

## Never animate during SSR

```jsx
/* Wrong: runs on the server; window/document undefined; no cleanup */
gsap.to(".hero", { y: 0 });

/* Right: effect-scoped, client-only, auto-cleaned */
useGSAP(() => { gsap.from(".hero", { y: 40, opacity: 0 }); }, { scope: root });
```

Guard any direct global access with `typeof window !== "undefined"`, or keep it inside an effect — effects never run on the server.

## Hydration safety

The server and first client render must produce identical markup, or React logs a hydration mismatch and may discard the tree — taking your animation setup with it.

- **Do not vary initial markup by viewport, device, or `prefers-reduced-motion` during render.** The server cannot know any of them. Apply those decisions in an effect, or with CSS media queries, which need no JS.
- Set the pre-animation hidden state via a class added in an effect (or via CSS), not by a render-time branch.
- Random or time-based initial values (stagger seeds, `Math.random()` offsets) differ between server and client. Compute them in an effect.

## StrictMode double-invocation

React StrictMode intentionally mounts, unmounts, and remounts components in development. **If your animation cannot survive that, it is broken — the mode is doing its job.** This is the mechanism behind most double-firing reports.

```jsx
/* Wrong: two ScrollTriggers, two listeners, after StrictMode remount */
useEffect(() => {
  ScrollTrigger.create({ trigger: ref.current, onEnter: reveal });
}, []);

/* Right: cleanup makes remounting harmless */
useEffect(() => {
  const st = ScrollTrigger.create({ trigger: ref.current, onEnter: reveal });
  return () => st.kill();
}, []);

/* Better in React: useGSAP reverts everything it created */
useGSAP(() => {
  ScrollTrigger.create({ trigger: ref.current, onEnter: reveal });
}, { scope: ref });
```

Never "fix" double-firing with a module-level `initialised` flag. It masks the leak, then breaks legitimate remounts — after navigation, the animation silently never runs again.

## useEffect vs useLayoutEffect

| Hook | When | Why |
| --- | --- | --- |
| `useLayoutEffect` | Setting an initial hidden state, or measuring before paint | Runs before paint — no flash of the un-animated state |
| `useEffect` | Everything else, including scroll triggers | Does not block paint |
| `useGSAP` | GSAP work in React | Wraps `useLayoutEffect` and adds context cleanup |

`useLayoutEffect` warns during SSR. `useGSAP` handles this; otherwise gate it or accept `useEffect` plus a CSS-set initial state.

## Measure after layout settles

Scroll positions and element sizes recorded at mount are wrong if layout changes afterwards. The usual culprits are web fonts and images.

```jsx
useGSAP(() => {
  const setup = () => {
    ScrollTrigger.create({ /* … */ });
    ScrollTrigger.refresh();
  };
  document.fonts?.ready.then(setup) ?? setup();
}, { scope: root });
```

Give images explicit `width`/`height` or `aspect-ratio` so they reserve space before loading — otherwise every trigger below them shifts. (Layout stability is `scribeo-ux-engineering`'s standard; this is the motion consequence of getting it wrong.)

## Route transitions

On navigation, the outgoing component's effects clean up and the incoming component's run. Anything not registered in that lifecycle leaks.

- Scroll position resets — or does not — depending on the router. Refresh scroll-linked animations after navigation.
- Global singletons (Lenis, a GSAP ticker callback) belong in one provider mounted once at the layout level, **not** per page. Creating Lenis per route produces stacked wheel handlers.
- Destroy on route change what was created on route entry. A ScrollTrigger pinned to a removed element throws or silently mispositions.
- Prefetching may mount components early. Never assume a mount means the element is visible.

```jsx
// Lenis as a single app-level provider
useEffect(() => {
  const lenis = new Lenis({ autoRaf: false });
  lenis.on("scroll", ScrollTrigger.update);
  const raf = (t) => lenis.raf(t * 1000);
  gsap.ticker.add(raf);
  gsap.ticker.lagSmoothing(0);
  return () => { gsap.ticker.remove(raf); lenis.destroy(); };
}, []);
```

## Refs, not selectors

Prefer refs, or scope selector strings to a root element. Global selectors match every instance of a component on the page — so a second card animates the first one's children.

```jsx
useGSAP(() => {
  gsap.from(".item", { y: 20, opacity: 0, stagger: 0.05 });
}, { scope: root });   // ".item" resolves only inside root
```

## Checklist

- [ ] `"use client"` on animating components; boundary kept tight
- [ ] No DOM access outside effects
- [ ] Initial markup identical on server and client
- [ ] Survives StrictMode mount/unmount/remount with no duplication
- [ ] Every animation, trigger, listener and ticker callback torn down where created
- [ ] Selectors scoped to a root, or refs used
- [ ] Refreshed after fonts and images settle
- [ ] Global scroll systems mounted once at layout level
- [ ] Verified after navigating away and back
