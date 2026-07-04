# 🌀 Flappy Sanic: True Dreamcast MIDI Engine

A nostalgic, high-fidelity browser game that fuses the iconic **Sega Dreamcast Startup Sequence** with a custom high-performance **Flappy Sanic** gameplay loop. Built entirely in vanilla JavaScript using the HTML5 Canvas API and powered by a highly optimized, distortion-free **Tone.js MIDI scheduler**.

---

## 🕹️ Live Demo Architecture

1. **The Retro Console Overlay**: Press the physical power button on the 3D-shaded Sega Dreamcast chassis to trigger the system boot.
2. **The 1:1 Boot Sequence**: Watch the signature orange swirl render dynamically with trigonometric vectors accompanied by full multi-voice stereo audio synthesis, followed by the classic grey SEGA license screen.
3. **Flappy Sanic Run**: Guide Sanic through Green Hill Zone while a fully hardcoded, multi-track MIDI synth engine plays the legendary soundtrack live in your browser.

---

## 🔊 Technical Audio Engineering (Tone.js Context Stability)

Traditional Web Audio setups often crash or get muted by browser safety heuristics when handling heavy polyphonic MIDI tracks alongside frequent gameplay sound effects. This project achieves 100% runtime stability using a dedicated architecture:

* **Persistent Global Synthesizers**: Instead of instantiating new audio nodes on every flap or coin collection—which triggers garbage-collection overhead and overlapping schedule bottlenecks—the game uses a persistent, globally scoped `sfxSynth` node.
* **Auto-Resuming Audio Context**: Every user interaction dynamically queries `Tone.context.state`. If the browser suspends the audio engine due to unexpected track stacking, the user input instantly fires `.start()` or `.resume()` inline before executing sound events.
* **Master Channel Headroom Buffer**: Individual instrument tracks are dynamically mixed down to an explicit `-18dB` threshold, preventing severe phase-summation clipping and blocking the browser from auto-muting the destination node.

---

## 🚀 Built With

* **HTML5 Canvas API** – High performance 60FPS retro sprite rendering, dynamic parallax background motion, and math-driven vector layouts.
* **Tone.js (v14.8.49)** – Clean sub-millisecond audio scheduling, square/triangle wave oscillators, and polyphonic sound synthesis.
* **@tonejs/midi** – Client-side binary parsing of compiled format 1 standard MIDI files.

## 🛠️ Customization

To swap out the background soundtrack, simply replace the placeholder string at the top of the runtime script:
```javascript
const base64MidiData = "YOUR_BASE64_MIDI_STRING_HERE";
