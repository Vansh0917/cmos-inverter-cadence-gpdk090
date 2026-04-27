# CMOS Inverter in Cadence Virtuoso using GPDK090

This repository documents the complete flow to design and simulate a CMOS inverter in Cadence Virtuoso using the **gpdk090** technology library.  
All work was done inside a VMware Linux environment, starting from launching Virtuoso up to DC and transient analysis of the inverter.

> This README is written as a step‑by‑step tutorial, with screenshots for each major step.  
> You can follow it to reproduce the project from scratch in your own Cadence setup.

---

## 1. Project Overview

- **Goal:** Design, simulate, and document a CMOS inverter using gpdk090 in Cadence Virtuoso.
- **Environment:** VMware virtual machine running Linux with Cadence IC 6.1.7 and Spectre.
- **Technology:** `gpdk090` generic 90 nm PDK.
- **Library:** Custom library `Vansh_lib`.
- **Analyses performed:**
  - DC sweep to obtain **Voltage Transfer Characteristic (VTC)**.
  - Transient analysis with a pulse input to verify switching behavior.

Supporting handwritten notes and raw screenshots are stored in the `notes/` folder.

---

## 2. Tools and Technology

- **VMware** – to run the Linux environment where Cadence is installed.
- **Cadence Virtuoso 6.1.7** – schematic, symbol, testbench and simulation environment.
- **Spectre** – simulator used via ADE L / Virtuoso Visualization & Analysis.
- **gpdk090** – generic 90 nm technology library attached to the custom design library.

---

## 3. Step‑by‑Step Design Flow

### 3.1 Launching Virtuoso inside VMware

1. Boot the **VMware** virtual machine.
2. Open a **terminal** inside the Linux guest.
3. Run:

   ```bash
   virtuoso &
   ```

4. Wait for Virtuoso to start and show the **Log / CIW** window.

```markdown

```

---

### 3.2 Creating a New Library (`Vansh_lib`)

1. From the CIW, open **Tools → Library Manager**.
2. In Library Manager, go to **File → New → Library…**.
3. Enter:
   - **Library Name:** `Vansh_lib`
   - **Path:** (default home path or custom path)
4. Choose **Attach to an existing technology library** when prompted.
5. Click **OK**.

```markdown

```

---

### 3.3 Attaching Technology: `gpdk090`

1. In the **Attach Library to Technology Library** form:
   - **Library:** `Vansh_lib`
   - **Technology Library:** `gpdk090`
2. Click **OK** to attach.

Virtuoso will confirm that `Vansh_lib` was successfully attached to `gpdk090` in the log window.

```markdown

```

---

### 3.4 Creating the CMOS Inverter Schematic

1. In Library Manager:
   - Library: `Vansh_lib`
   - Cell: `inverter`
   - View: `schematic`
2. Click **File → New → CellView…** or right‑click and choose **New**.
3. The **Virtuoso Schematic Editor** opens.
4. Use **Create → Instance** to place:
   - One **PMOS** (from gpdk090 devices)
   - One **NMOS** (from gpdk090 devices)
5. Connect them to form a standard CMOS inverter:
   - PMOS source to VDD, drain to output.
   - NMOS drain to output, source to GND.
   - Gates tied together as input.
6. Add pins:
   - `vin` (input), `vout` (output), `vdd`, `gnd`.
7. **Check and Save** the schematic.

Your transistor sizing can follow a simple example such as:
- NMOS: L = 100 nm, W = 1 µm.

```markdown

```

---

### 3.5 Generating the Inverter Symbol

1. In the schematic window, use **Create → Cellview → From Cellview…**.
2. Source view: `schematic`, Target view: `symbol`.
3. Accept default pin arrangement or customize:
   - Top: `vdd`
   - Bottom: `gnd`
   - Left: `vin`
   - Right: `vout`
4. Click **OK**, then save the symbol.

```markdown

```

---

### 3.6 Building the Testbench for Inverter

1. Create a new cell, for example:
   - Library: `Vansh_lib`
   - Cell: `inverter_tb`
   - View: `schematic`
2. Place the **inverter symbol** from `Vansh_lib`.
3. Add:
   - **VDD source:** `vdc` = 1.8 V (supply).
   - **Input source:** `vpulse` with:
     - `V1 = 0 V`
     - `V2 = 1.8 V`
   - Connect grounds to global `gnd`.
   - Optionally add a small **load capacitor** from `vout` to `gnd`.
4. Label nets: `vin`, `vout`, `vdd`, `gnd`.
5. **Check and Save** the testbench schematic.

```markdown

```

---

## 4. Simulation Setup

Simulations are done in **ADE L** with Spectre as the simulator, using the `inverter_tb` schematic.

### 4.1 Opening ADE L

1. With `inverter_tb` schematic open, choose:
   - **Launch → ADE L**.
2. Ensure **Spectre** is selected as the simulator.

---

### 4.2 Transient Analysis Setup

Initial mistake (learning point):  
- Stop time was set to **500 ms**, which is **500,000,000 ns**, making simulation extremely slow.

Final correct setup:

- **Analysis:** `tran` (transient)
- **Stop time:** `200n` (200 ns)
- **Input pulse (vpulse)**:
  - `V1 = 0`
  - `V2 = 1.8`
  - Reasonable **period** example: `20n` (10 ns high, 10 ns low)
  - Rise/Fall time: small values like `50p` or `100p`.

Make sure `vin` and `vout` are selected to be saved/printed.

```markdown

```

---

### 4.3 Running Transient Analysis

1. In ADE L, click **Netlist and Run**.
2. After simulation completes, open the waveform window (Virtuoso Visualization & Analysis).
3. Plot:
   - `vin` (input pulse)
   - `vout` (inverter output)

Expected behavior:  
- When `vin` is low (≈0 V), `vout` is high (≈1.8 V).  
- When `vin` is high (≈1.8 V), `vout` is low (≈0 V).

```markdown

```

---

### 4.4 DC Analysis Setup (Voltage Transfer Characteristic)

1. In ADE L, choose **Analyses → Choose**.
2. Select **dc** analysis:
   - **Sweep Variable:** component parameter (the DC value of the input source).
   - **Start:** `0`
   - **Stop:** `1.8`
   - **Increment:** small step such as `0.01` or similar.
3. Ensure both `vin` and `vout` are selected in the Outputs list (so that both appear in the DC plot).

```markdown

```

---

### 4.5 Running DC Analysis

1. Click **Netlist and Run** in ADE L again with DC analysis enabled.
2. In the waveform viewer, switch to the **DC dataset**.
3. Plot `vout` vs input voltage to get the **VTC** of the inverter.

```markdown

```

---

## 5. Results and Observations

- **Transient analysis:**
  - The inverter correctly inverts the input pulse.
  - With a stop time of **200 ns**, several input transitions can be observed clearly without long simulation times.

- **DC analysis:**
  - The VTC shows the typical CMOS inverter transfer curve.
  - Output is high when input is low, and goes low as input approaches VDD.
  - The switching region (where the output transitions) is near the mid‑supply region, around the threshold voltage combination of PMOS/NMOS.

```markdown

```

- **Important learning:**
  - Setting an excessively large stop time (like 500 ms) slows down simulations dramatically.
  - Adjusting the stop time to match the real signal period (200 ns in this case) makes simulations fast and useful.

---

## 6. Supporting Documents

This repository can also include:

- `notes/Screenshot-2026-04-25-002055.pdf` – raw environment and simulation log screenshots.
- `notes/CMOS-inverter-Cadence-virtuoso-2.pdf` – handwritten project notes and flow.

These are helpful for anyone who wants to see the original working environment and notes.

---

## 7. Video References

The following YouTube playlist was followed during the project:

1. **Cadence Virtuoso – Inverter Design Intro**  
   `https://www.youtube.com/watch?v=sOqgcOgouKk&list=PL86shurD5y344qr_c7b_o8jC3ucKfStmY`

2. **Symbol and Testbench Construction**  
   `https://www.youtube.com/watch?v=TcCHsO5j9CI&list=PL86shurD5y344qr_c7b_o8jC3ucKfStmY&index=2`

3. **DC and Transient Analysis of Inverter**  
   `https://www.youtube.com/watch?v=moXKwAzmxXU&list=PL86shurD5y344qr_c7b_o8jC3ucKfStmY&index=3`

4. **Additional Inverter Examples and Variations**  
   `https://www.youtube.com/watch?v=4rMDzVLAAXo&list=PL86shurD5y344qr_c7b_o8jC3ucKfStmY&index=4`

You can also place these links into `references/video-links.md`.

---

## 8. How to Re‑Use or Extend This Project

To reuse or extend this project:

1. Recreate the **library + gpdk090** setup in your Cadence installation.
2. Follow the schematic, symbol, and testbench steps above.
3. Once the inverter is working, extend to:
   - 2‑input NAND gate
   - 2‑input NOR gate
   - 3‑ or 5‑stage ring oscillator using the inverter

For each new cell, follow the same documentation style: schematic → symbol → testbench → simulation → screenshots.

---

## 9. Author

**Vansh Vaghela**  
Electronics & Communication (VLSI) student, interested in ASIC, FPGA and custom IC design flows.
