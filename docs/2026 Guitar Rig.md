# 4CM + W/D/W Electric Guitar Rig Architecture
## Final Definitive Engineering Blueprint

This document serves as the absolute single source of truth for the signal routing, preset logic, and custom hardware integration of a hybrid 4-Cable Method (4CM) and Wet/Dry/Wet (W/D/W) switching system. 

### System Design Overview
The rig is physically optimized for a **Temple Audio Duo 34** pedalboard and a **4U Gator rolling rack bag**. 

The system logic is controlled via a **Carl Martin Octa-Switch "The Strip"**. Because the Octa-Switch transmits fixed, non-configurable MIDI Program Change (PC) commands mapping strictly to the active preset bank (e.g., Preset 1 sends PC 1, Preset 2 sends PC 2), the system topology relies on mapping the corresponding patches directly inside the **Boss SDE-3000D** dual digital delay to mirror these incoming integers.

---

## Part 1: Main System Signal Flow & Bus Topography

The audio signal flow is divided into distinct functional blocks to optimize dynamic tracking, isolate high-gain preamplification, and distribute an expansive stereo image while maintaining a punchy, phase-coherent center dry signal.

### 1. The Front-End Tracking Engine
*   **Input Stage:** Guitar $\rightarrow$ **D'Addario Tuner** Input.
*   **Gain Dynamics:** D'Addario Tuner Output $\rightarrow$ **Boss FV-30 Volume Pedal** Input $\rightarrow$ **JHS Buffered Splitter** Input.
*   **Parallel Tracking Path (Output A):** Connects to the **Amazon Basics Compressor** Input $\rightarrow$ Compressor Output $\rightarrow$ **Noise Gate Key/Input** jack. 
    *   *Technical Design Note:* This architecture uses a dedicated clean tracking line to heavily increase the gate's dynamic tracking sensitivity without compressing the audible audio path.
*   **Main Audio Path (Output B):** Connects directly into the **Carl Martin Octa-Switch "The Strip"** Main Input to drive the core effects blocks.

### 2. Switcher Loop Allocation Matrix

| Loop | Hardware Unit | Audio Path Context | Preset Logic / Governance |
| :--- | :--- | :--- | :--- |
| **Loop 1** | Aion FX Convex | Dinosaural OTC-201 Clone Parallel Opto-Compressor. | As needed for clean/rhythm dynamics. |
| **Loop 2** | GUPTech Le Chiou | Tight Overdrive / Distortion voicing. | Enabled for mid-to-high gain staging. |
| **Loop 3** | *Empty* | Open spare loop for expansion. | Permanently bypassed. |
| **Loop 4** | **AMT Stonehead SH-100-4R** | Preamp Stage (Send $\rightarrow$ Front Input; Head FX Loop 1 Send $\rightarrow$ Switcher Return). | **ON** for all presets using the core preamp. |
| **Loop 5** | TC Electronic Quintessence | Pitch Harmonizer (Mono-to-Mono infrastructure). | **ON** exclusively for dual-guitar solo tracking; **OFF** for rhythm. |
| **Loop 6** | Hardwire Noise Gate | Clamping engine (Send $\rightarrow$ Gate Input; Gate Output $\rightarrow$ Return). | **ON** for high-gain patches to clamp preamp/harmony hiss; **OFF** for cleans. |
| **Loop 7** | **DIY Custom Isolation Box** | Parallel W/D/W Split & Ground Isolation Engine (See Part 3). | **PERMANENTLY ON** to sustain dry amp feed and downstream routing. |
| **Loop 8** | **Boss SDE-3000D** | Ambient Space & Wet Stereo Imaging Stage. | **ON** for all ambient, delay, and stereo chorus configurations. |

### 3. Downstream Wet/Dry Separation
*   **Dry Output Path:** Extracted out of the **DIY Box Direct Out** $\rightarrow$ Switcher Loop 7 Return to complete the internal trace. The final **DIY Box Isolated Out** travels down the 8-channel snake to the **AMT Head FX Loop 1 Return**, feeding the fully harmonized, gated dry tone directly to the center **2x12 Cabinet** while completely shattering the ground loop hum.
*   **Wet Output Path:** Inside Loop 8, a mono TS patch cable feeds the **Boss SDE-3000D Input L/Mono** (split internally). The **Boss SDE-3000D Outputs (L & R)** output via dual mono TS cables into a **Dual-TS-to-TRS Insert Cable** back to the Loop 8 Return. 
*   **Power Amplification:** The Strip's Master Outputs 1 & 2 connect via the snake to the **ISP Technologies Stealth Pro Inputs 1 & 2** $\rightarrow$ driving the Left and Right **Custom 1x12 Monitor Wedges**.

---

## Part 2: Hardware Settings & Rack Processing

### 1. AMT Stonehead FX Loop 2 (Dry Processing Bus)
To shape the central dry projection without degrading or phase-cancelling the wet stereo delay trails, secondary post-preamp processing is isolated strictly to the dry backline enclosure:
$$\text{AMT FX Loop 2 Send} \longrightarrow \text{Parametric EQ Input} \longrightarrow \text{BBE Sonic Maximizer Input} \longrightarrow \text{AMT FX Loop 2 Return}$$

### 2. Boss SDE-3000D Digital Parameters
The digital processor must be configured globally and per patch to prevent dry signal leakage and maintain spatial independence.

*   **Direct Mute (Kill-Dry):** `DIR.MUTE` must be set to `ON` inside the `[SETUP] -> MASTER` utility menu. This locks out any duplicate dry electric signal from leaking into the wet wedges, ensuring 100% studio-grade physical separation.
*   **Internal Routing Structure:** Set `STRUCT` to `PARA 2` (Parallel 2) in the setup menu. This isolates Delay Engine 1 and Delay Engine 2 so they simultaneously receive the isolated input signal and process their respective stereo fields independently.
*   **Dual Stereo Preset Configuration (Chorus + Delay Patches):**
    *   **Engine 1 (Stereo Chorus Role):** Configure `TIME` parameter between `10 ms` and `25 ms`. Enable LFO modulation (`MOD`), dialing a slow rate and deep depth. Toggle `PHASE` (Output Phase Reverse) to `ON` on a single channel to achieve an ultra-wide psychoacoustic chorus spread.
    *   **Engine 2 (Stereo Delay Role):** Configure traditional rhythmic reflections (e.g., `350 ms`) with feedback and wet mix levels balanced to taste.

---

## Part 3: DIY 3-Connector Isolation Box Build (Triad TY-250P)

By utilizing three audio jacks instead of two, the Triad TY-250P audio transformer circuit handles the parallel signal split natively inside a single shielded project enclosure.

```text
               CUSTOM ENCLOSURE INTERNAL CIRCUIT SCHEMA

         INPUT JACK                        DIRECT OUTPUT JACK
      (Standard Metal)                      (Standard Metal)
    +-----------------+                  +--------------------+
    |                 |                  |                    |
    |   TIP (Signal)  |---+----------->  |   TIP (To Loop 7   |
    |                 |   |              |        Return)     |
    |                 |   |              |                    |
    | SLEEVE (Ground) |---+----------->  | SLEEVE (Ground)    |
    +-----------------+   |              +--------------------+
          |               |                         |
   [Bolted to Case]       |                  [Bolted to Case]
          |               |                         |
          +---------------+                         |
          |                                         |
          v                                         v
   (Acts as RF Shield)                      (Shared Box Shield)
          |
          +-------> [ Pin 2 ] (Primary +)
          +-------> [ Pin 4 ] (Primary -)
 
 ==========================================================================
                     [ TRIAD TY-250P TRANSFORMER COILS ]
                      * Pins 1 & 3 are left unconnected
                      * Jumper wire connects Pin 6 directly to Pin 7
 ==========================================================================

                           [ Pin 5 ] (Secondary +)
                           [ Pin 8 ] (Secondary -)
                               |
                               v
                  +--------------------------+
                  |   DPDT PHASE REVERSE     |
                  |     SWITCH ENGINE        |
                  +--------------------------+
                               |
                               v
                     ISOLATED OUTPUT JACK
                    (Cliff / Nylon Washers)
                  +--------------------------+
                  |  TIP (To Phase Switch)   |
                  |                          |
                  |  SLEEVE (To Phase Switch)|
                  +--------------------------+
                               |
                     [Completely Insulated] (Breaks the Ground Loop)
```

### DPDT Phase Inversion Switch Schematic
```text
  TOP ROW:      [ Pin 1 ] ──(Crisscross Jumper)── [ Pin 6 ]
                   │                                 │
                   └─► To Isolated Jack TIP          │
                                                     │
  MIDDLE ROW:   [ Pin 2 ] ◄── From Trans. Pin 5      │
                [ Pin 5 ] ◄── From Trans. Pin 8      │
                                                     │
  BOTTOM ROW:   [ Pin 3 ] ──(Crisscross Jumper)── [ Pin 4 ]
                   │
                   └─► To Isolated Jack SLEEVE
```
***

