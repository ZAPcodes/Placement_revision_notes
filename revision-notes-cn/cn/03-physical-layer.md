# 03 — Physical Layer ★

Moves raw **bits** over a medium as electrical, optical, or radio signals. Defines connectors, voltages, bit timing, and data rates.

## Transmission media

| | Type | Notes |
|---|---|---|
| **Guided** | Twisted pair (UTP/STP) | Cheapest; Ethernet LANs (Cat5e/Cat6); twisting cancels interference; ~100 m limit |
| | Coaxial | Cable TV, older Ethernet; better shielding than twisted pair |
| | Optical fibre | Light pulses; highest bandwidth, longest distance, immune to electromagnetic interference; single-mode (long distance) vs multi-mode (short) |
| **Unguided** | Radio waves | Omnidirectional; Wi-Fi, FM, mobile |
| | Microwaves | Line-of-sight, directional; satellite, point-to-point links |
| | Infrared | Short range, can't pass walls; TV remotes |

## Signals and channel capacity
- **Bandwidth (Hz):** range of frequencies the channel passes. **Data rate (bps):** bits per second.
- **Nyquist (noiseless channel):** `C = 2 × B × log₂(L)` where B = bandwidth (Hz), L = number of signal levels.
- **Shannon (noisy channel):** `C = B × log₂(1 + SNR)` — the theoretical upper limit; SNR as a **ratio**, not dB.
- **SNR in dB** = `10 × log₁₀(SNR)`. So 30 dB → SNR = 1000.

> [!WARNING]
> **Tricky:**
> - Convert dB to a ratio before using Shannon: 30 dB is 1000, not 30.
> - Shannon gives the **maximum possible** rate; Nyquist tells you how many **levels** you'd need to get there. Typical question: find Shannon's C, then use Nyquist to find L.
> - Increasing levels (L) raises data rate under Nyquist, but more levels are harder to distinguish in noise — Shannon caps it.

**Worked example:** B = 3000 Hz, SNR = 30 dB → SNR = 1000. C = 3000 × log₂(1001) ≈ 3000 × 9.97 ≈ **29.9 kbps**.

## Line coding (digital data → digital signal)
- **NRZ (Non-Return to Zero):** high = 1, low = 0. Simple, but long runs of 0s/1s lose clock sync and create a DC component.
- **NRZ-I:** a transition means 1, no transition means 0.
- **Manchester:** every bit has a transition in the middle (e.g. low→high = 1). **Self-clocking**, no DC component, but needs **twice the bandwidth**. Used in classic 10 Mbps Ethernet.
- **Differential Manchester:** always a mid-bit transition; a transition at the *start* of the bit encodes 0 (or 1, by convention).

**Baud rate vs bit rate:** baud = signal changes per second; bit rate = baud × bits per signal element. Manchester: baud = 2 × bit rate.

## Multiplexing (sharing one link among many signals)
- **FDM (Frequency Division):** each signal gets its own frequency band (radio, cable TV). Analog.
- **TDM (Time Division):** each signal gets time slots in turn. **Synchronous TDM** gives fixed slots (wasted if idle); **statistical TDM** assigns slots on demand.
- **WDM (Wavelength Division):** FDM for fibre — different colours (wavelengths) of light.
- **CDMA (Code Division):** all share the same frequency and time, separated by unique codes (3G mobile).

## Physical-layer devices
- **Repeater:** regenerates (not just amplifies) a weakened signal to extend distance. 2 ports.
- **Hub:** multi-port repeater — whatever arrives on one port is sent out **all** others.

> [!WARNING]
> **Tricky:** a hub creates **one collision domain** and **one broadcast domain** for all connected devices; it's half-duplex and has no idea what a MAC address is. That's why switches replaced hubs.
