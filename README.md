# Scalpel

**A "Proton for Android."** An inject-don't-fork optimizer for Winlator-class x86
Windows-game emulation — make stock Wine / Box64 / DXVK / Turnip faster by
*injecting* targeted fixes, without forking any of them.

> The surgeon that operates on the emulation stack without removing the organ.

> [!NOTE]
> This is a **public showcase repo** — write-up and a small in-browser viewer.
> The implementation (the Vulkan layer, attributor, and build tooling) lives in a
> private repo. There is no buildable source here by design.

---

## The story

It started as a crash-out. I spent a day trying to make an Android phone run a 2017
PC game faster, watched every config idea I had die on contact with reality, and
walked away with one number that reframed the whole problem.

→ **[I am Crashing out (a WinEmulation Story)](https://thanukamax.github.io/lab/writing/the-gpu-wasnt-even-trying)**
— the origin story, written while I was still annoyed.

The number: a flagship **Adreno 750 sitting at 99.8% busy, 37% clock, 38fps** on
*NieR: Automata*. That's roughly **~10× the work native hardware would do** for the
same frame. The bottleneck isn't a setting you forgot to flip — it's **structural
translation overhead** baked into the emulation stack itself.

I swept five config levers (GOS, Turnip version, resolution, env-flags, DXVK
version). All flat, regressive, or wouldn't launch. **Config knobs don't move it.**

## Why "inject, don't fork"

Two dead ends frame the right path:

- **Forking everything = drowning.** Wine, Box64, DXVK, and Turnip are millions of
  lines maintained by expert teams. A full fork loses their ongoing fixes and is
  unmaintainable solo.
- **Config tuning = doesn't move the needle.** Proven above.

The third path is the one **Valve's Proton** already walks for desktop: stock
Wine + DXVK, a thin patch-set, a per-game fix database, and a version manager —
"just update Proton." There is **no good equivalent on Android.** Winlator is closer
to raw Wine. That's the white space.

So Scalpel keeps every dependency **stock and auto-updating**, and ships only the
**delta**:

1. Dependencies stay stock + auto-updating.
2. Fixes ride on top through **sanctioned injection points** — not hacks.
3. We maintain only the injection layer + a minimal, auto-rebased patch-overlay.
4. **Update = update the app.** Never the deps.

## The surgeon's kit

| Target | Mechanism (no fork) | Prior art |
|---|---|---|
| **Turnip / GPU** | Vulkan layer above the driver — barrier elision, pipeline pre-cache, present pacing | vkBasalt, MangoHud, Fossilize |
| **DXVK / D3D** | wrapper DLL + `dxvk.conf` injection | dxvk-async |
| **Wine** | DLL overrides + LD_PRELOAD + per-game fix DB | Proton `protonfixes` |
| **Box64 / CPU** | env tuning + patch-overlay for deep reach | — |
| **Patch-overlay** | minimal diffs auto-rebased onto upstream releases | git patch series + CI |
| **Dep/version manager** | keep deps current with our layer applied; ship as the app | Winlator `.wcp`, Proton runtime |
| **The attributor** | per-frame blame → tells us *what* to inject | — |

### The honest reach limit

| Reachable by injection (no fork) | Needs the patch-overlay |
|---|---|
| barrier elision, call batching | Box64 dynarec codegen |
| pipeline / shader pre-caching | Turnip's command encoder |
| present pacing, frame limiting | Wine syscall-translation core |
| per-game config / DLL workarounds | deep dispatch internals |

## The two halves are one project

- **Attributor = the targeting computer.** Measures a frame and says "*this* call
  costs 8ms." Without it, you inject blind.
- **Injection framework = the delivery gun.** Ships the fix the attributor found,
  without forking anything.
- **The app = the product.** Version-manager + fix-DB + the layer, updatable as one
  unit.

Speed is the whole point, and I treat time-to-result as a first-class feature — same
discipline as the rest of my tools: **[Treating latency as a feature](https://thanukamax.github.io/lab/writing/treating-latency-as-a-feature)**.

## Status

- **M1 — built.** A pass-through Vulkan layer that loads under Winlator (the GPU
  injection point works, no root) + a per-frame attributor (CPU frametime vs
  GPU-busy ms → the residual that names the bottleneck regime).
- **M2 — in progress.** Barrier/render-pass profiling and the **first injected
  fixes** (depth/stencil store elimination, attachment-op knob matrix), A/B'd on a
  real game.

### Roadmap

| Phase | What |
|---|---|
| **P0** | De-risk — does a Vulkan layer even load under Winlator? no-root attributor method? auto-rebase overlay? |
| **P1** | Attributor MVP — per-frame attribution on a real game |
| **P2** | First injected fix — the layer + the *specific* fix the attributor found |
| **P3** | Fix DB + dep manager + packaging |
| **P4** | The updatable app |

## Read more

- **Blog:** [thanukamax.github.io/lab](https://thanukamax.github.io/lab/)
- **Origin story:** [I am Crashing out (a WinEmulation Story)](https://thanukamax.github.io/lab/writing/the-gpu-wasnt-even-trying)
- **On speed:** [Treating latency as a feature](https://thanukamax.github.io/lab/writing/treating-latency-as-a-feature)

---

*Scalpel is a working codename. Built by [@Thanukamax](https://github.com/Thanukamax).*
