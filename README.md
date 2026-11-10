# TB Clockwork Concubine

A VST 2.4 plugin model of the HH Electronics Clockwork Concubine flanger,
built from its schematic (drawn by Dirk Hendrik, rev A, 2007/2008, in `Docs/`).

- 64-bit Intel, **Mac OS X 10.8 and later**, made for Pro Tools 11 through Blue Cat PatchWork
- Built and tested on GitHub Actions; the build fails unless the binary declares
  10.8 as its minimum and links only libraries that ship with 10.8
- Personal, non-commercial use

## Controls

| # | Name | Control | Range |
|---|------|---------|-------|
| 1 | Freq | FREQUENCY (VR2 1M rev log): LFO rate | 0.1 – 10.5 Hz |
| 2 | Depth | DEPTH (VR1 100K lin): how far the delay sweeps | 0 – 10 |
| 3 | Reso | RESONANCE (VR4 10K lin): feedback | 0 – 10 |
| 4 | Effect | The S1 footswitch: effect in, or straight through | In / Out |
| 5 | Input | *plugin only*: level into the pedal (effect path) | ±12 dB |
| 6 | Output | *plugin only*: output trim | ±24 dB |

The pedal is mono. In stereo, each side runs through its own copy of the
circuit with the same settings and the two sweeps in step; a mono source is
processed once.

## What's modelled

From the drawing:
- the Q1 input buffer (C4 100n / R14 100K, C8 6u8 into R19 100K)
- IC2 with the 47K-47K T network (4K7 + 4n7): about +15 dB of treble lift
  above 1.2 kHz going into the BBD
- the RESONANCE loop: the wet signal through RX1 6K8, the C18 47n + R37 470 treble cut,
  the VR4 track, and C5 4n7 / R20 100K into IC2's + input (which cuts the feedback
  below about 340 Hz)
- the SAD1024 with both 512-stage sections in parallel, clocked half a period
  apart and their outputs joined: a sample-and-hold bucket chain with its
  staircase output and clock aliasing, loaded by R32 10K / C13 10n, then C16 47n
- the IC5 mixer: dry from IC2 through R31 47K, wet through the RX2 10K / CX 47n +
  R39 2K2 / R40 22K de-emphasis, R33 22K || CX2 6n8 feedback; C19 / R38 out
- the 741s on the ±6V8 zener rails (output swing, slew rate)
- S1: Effect Out passes the input straight through

What that gives: with the knobs at the defaults, the dry and wet signals are
nearly equal from about 300 Hz to 1 kHz, so the notches there are deep (about
-18 dB). Below about 300 Hz the wet signal is cut (C16 and the de-emphasis), so
the bass stays dry and solid, and the wet signal is darker than the dry. The
dry path through the pedal is about 6.6 dB down (R33 / R31). The RESONANCE loop
as drawn tops out at about +5.5 dB peaks, so the pedal doesn't self-oscillate.

Parts marked X on the drawing (RX1, RX2, CX, CX2) were probably chosen on test;
they're the first values to check if the real pedal sounds brighter or more
resonant (`Source/DSP/Flanger.cpp`).

Not fixed by the drawing, so estimated (named constants in `Source/DSP/Flanger.cpp`):
- **the delay range**: the clock oscillator (IC1.A, Q2, C1 470p, IC1.D) is set so
  the BBD delay is 3 ms with DEPTH at 0 and sweeps 0.45 – 5.6 ms at full DEPTH
  (`kDelay0`, `kSweep`); the delay follows the LFO in a straight line, as the
  ramp-and-compare clock does
- **the LFO**: a triangle (IC1.B integrator); 0.1 Hz fully anticlockwise, the top
  end set by the ratio of VR2 to the 15K / 27K network (`kLfoSlowHz`)
- **the SAD1024**: gain 0.9, output resistance 1K5 into C13, clipping near
  1.6 V peak with VR3 centred (`kBbdGain`, `kBbdRout`, `kBbdClip`)
- Q1 is drawn as a BC314 (PNP) with its collector to the positive rail; it's
  modelled as the unity-gain emitter follower it's meant to be

## The panel

On the Mac the plugin opens its own panel (480 × 720, the pedal's 2:3), laid out like the
pedal: the FREQUENCY, DEPTH and RESONANCE knobs (scales marked 0 and 10), the
name plate, and the striped panel with the stomp switch, which glows red with
the effect in. The strip along the bottom has the TB Electronic mark, Input,
Output, the presets and the readout. The drawing's lamps (LP1-3) are left out.

Drag knobs up/down (hold Shift for fine control), double-click a knob to reset
it, or use the scroll wheel. Click the stomp switch to switch the effect in
or out.

**Presets:** click SAVE, then a slot 1–8. Click a saved slot to recall it; the
slot in use lights green. Presets are shared by every instance, in
`~/Library/Application Support/TB/TB Clockwork Concubine/Presets.txt`.

## Install

1. GitHub → Actions → latest run → Artifacts → `TB-Clockwork-Concubine-mac-vst`
2. Unzip twice to get `TB Clockwork Concubine.vst`
3. Copy it to `/Library/Audio/Plug-Ins/VST/` and rescan in PatchWork

## Build and test

- `make -C Tests run` — preset store and DSP tests, demo WAVs in `Tests/renders`
- `make -C Tests vst-run` — loads the plugin in a mini VST host
- `make -C Tests asan` — both under AddressSanitizer / UBSan
- `scripts/build_vst_mac.sh` — the Mac bundle (CI runs this, plus the panel test,
  which publishes screenshots to the `gui-preview` branch)
