# CMOS Inverter in Cadence Virtuoso using GPDK090

This repository documents the complete flow to design and simulate a CMOS inverter in Cadence Virtuoso using the **gpdk090** technology library.  
The work is done inside a VMware Linux environment, starting from launching Virtuoso up to DC and transient analysis of the inverter.

> The README is written as a step‑by‑step guide.  
> You can follow it to reproduce the project in your own Cadence setup.

---

## 1. Project Overview

- **Goal:** Design, simulate, and document a CMOS inverter using gpdk090 in Cadence Virtuoso.
- **Environment:** VMware virtual machine running Linux with Cadence IC 6.1.7 and Spectre.
- **Technology:** `gpdk090` generic 90 nm PDK.
- **Library:** Custom design library `Vansh_lib`.
- **Analyses performed:**
  - DC sweep to obtain the **Voltage Transfer Characteristic (VTC)**.
  - Transient analysis with a pulse input to verify switching behavior.

Additional handwritten notes and raw screenshot logs are included in the `notes/` folder for reference.

---

## 2. Tools and Environment

- **VMware** – to run the Linux environment where Cadence is installed.
- **Cadence Virtuoso 6.1.7** – schematic editor, symbol editor, and analysis environment.
- **Spectre** – simulator invoked from ADE L / Virtuoso Visualization & Analysis XL.
- **gpdk090** – generic 90 nm process design kit attached to the design library.

---

## 3. Design Flow

### 3.1 Launching Virtuoso inside VMware

1. Boot the **VMware** virtual machine.
2. Open a **terminal** in the Linux guest.
3. Start Virtuoso:

   ```bash
   virtuoso &
   ```

4. Wait for the **CIW (Command Interpreter Window)** and other Virtuoso windows to appear.

```markdown

```

---

### 3.2 Creating the Design Library (`Vansh_lib`)

1. From the CIW, open **Tools → Library Manager**.
2. In Library Manager, go to **File → New → Library…**.
3. Fill in the fields:
   - **Library Name:** `Vansh_lib`
   - **Path:** Choose a suitable location (default home path is fine).
4. In the technology options, choose **Attach to an existing technology library**.
5. Click **OK**.

```markdown

```

---

### 3.3 Attaching `gpdk090` Technology

1. The **Attach Library to Technology Library** form opens.
2. Confirm:
   - **Library:** `Vansh_lib`
   - **Technology Library:** `gpdk090`
3. Click **OK**.

The CIW log should show a message that `Vansh_lib` has been successfully attached to `gpdk090`.

```markdown

```

---

### 3.4 Creating the CMOS Inverter Schematic

1. In Library Manager:
   - Library: `Vansh_lib`
   - Create a new cell, e.g. **Cell Name:** `inverter`
   - **View:** `schematic`
2. The **Virtuoso Schematic Editor** opens.
3. Use **Create → Instance** to place:
   - One **PMOS** device from the gpdk090 library.
   - One **NMOS** device from the gpdk090 library.
4. Connect them as a standard CMOS inverter:
   - PMOS source → VDD.
   - PMOS drain → output node (`vout`).
   - NMOS drain → output node (`vout`).
   - NMOS source → GND.
   - PMOS gate and NMOS gate tied together as input (`vin`).
5. Add pins:
   - `vin` (input)
   - `vout` (output)
   - `vdd` (supply)
   - `gnd` (ground)
6. **Check and Save** the schematic.

Device sizing (example):
- NMOS: length ≈ 100 nm, width ≈ 1 µm.
- PMOS: sized appropriately to achieve symmetrical switching (for learning, equal widths are fine).

```markdown

```

---

### 3.5 Generating the Inverter Symbol

1. With the `inverter` schematic open, go to **Create → Cellview → From Cellview…**.
2. Source view: `schematic`.
3. Target view: `symbol`.
4. Accept the default pin configuration or adjust:
   - Top: `vdd`
   - Bottom: `gnd`
   - Left: `vin`
   - Right: `vout`
5. Click **OK**, then **save** the symbol.

This symbol will now be used in a **test circuit** to verify the inverter.

```markdown

```

---

### 3.6 Creating the Test Circuit Using the Inverter Symbol

Instead of a separate “testbench tool”, a **test circuit schematic** is created that uses the inverter symbol and sources.

1. In Library Manager, create a new cell:
   - Library: `Vansh_lib`
   - Cell: e.g. `inverter_test`
   - View: `schematic`
2. Open this new schematic.
3. Place the **inverter symbol** from `Vansh_lib`.
4. Add sources and load:
   - **VDD source:** `vdc` with DC value `1.8 V`.
   - **Input source:** `vpulse` configured to toggle between 0 V and 1.8 V.
   - Optional: load capacitor from `vout` to `gnd` for realistic output loading.
5. Connect:
   - `vdd` pin of the symbol to VDD source.
   - `gnd` pin to ground.
   - `vin` pin to the `vpulse` source.
   - `vout` pin to a probe and optional load.
6. Label the nets clearly (`vin`, `vout`, `vdd`, `gnd`) and **Check and Save** the test circuit schematic.

```markdown

```

This schematic is the **test ckt** used to test the inverter symbol.

---

## 4. Simulation Setup

All simulations are run from the **test circuit schematic** using ADE L and the Spectre simulator.

### 4.1 Opening ADE L from the Test Circuit

1. With the `inverter_test` (or your chosen test-circuit cell) schematic open:
   - Go to **Launch → ADE L**.
2. Ensure the **Simulator** is set to **Spectre**.

---

### 4.2 Transient Analysis Setup

The transient analysis checks how the inverter output responds to a pulsed input.

**Input Pulse (`vpulse`) suggested parameters:**
- `V1 = 0 V`
- `V2 = 1.8 V`
- `Rise time` ≈ `50 ps – 100 ps`
- `Fall time` ≈ `50 ps – 100 ps`
- `Pulse width` and `period` chosen to see several cycles within a few hundred nanoseconds.

**Important note:**  
Initially, a stop time of **500 ms** was used, which is extremely large for a simple inverter and makes the simulation very slow. It was corrected to **200 ns**, which is suitable for observing digital switching behavior.

**Steps:**
1. In ADE L, open **Analyses → Choose…**.
2. Select **tran** (transient analysis).
3. Set:
   - **Stop time:** `200n` (200 ns)
   - Leave other settings default or as needed.
4. Click **OK**.
5. In the Outputs panel, choose `vin` and `vout` to be plotted/saved.

```markdown

```

---

### 4.3 Running Transient Analysis

1. Click **Netlist and Run** in ADE L.
2. After simulation finishes, the **Virtuoso Visualization & Analysis** window opens.
3. Plot:
   - `vin` – the input pulse.
   - `vout` – the inverter output.

**Expected behavior:**
- When `vin` is low (≈ 0 V), `vout` is high (≈ 1.8 V).
- When `vin` is high (≈ 1.8 V), `vout` is low (≈ 0 V).

```markdown

```

---

### 4.4 DC Analysis Setup (Voltage Transfer Characteristic)

DC analysis is used to obtain the **VTC** of the inverter by slowly sweeping the input from 0 V to 1.8 V.

**Steps:**
1. In ADE L, open **Analyses → Choose…**.
2. Select **dc** analysis.
3. For **Sweep Variable**, select the DC parameter of the input source (or the input node, depending on your setup).
4. Set:
   - **Start:** `0`
   - **Stop:** `1.8`
   - **Step:** a small increment (e.g. `0.01` or suitable value).
5. Click **OK**.
6. In Outputs, ensure `vin` and `vout` are selected.

```markdown

```

---

### 4.5 Running DC Analysis

1. With DC analysis enabled, click **Netlist and Run** again.
2. In the waveform viewer, select the **DC dataset**.
3. Plot `vout` vs the swept input.

The curve should show the inverter transitioning from high to low as the input rises from 0 V to 1.8 V, giving the typical S-shaped VTC.

```markdown

```

---

## 5. Results and Observations

- **Transient Analysis:**
  - The inverter output correctly inverts the pulse at its input.
  - Using **200 ns** as the stop time is sufficient to see multiple switching cycles without wasting simulation time.
  - Initial use of **500 ms** stop time demonstrated why very large time windows should be avoided for small digital cells.

- **DC Analysis:**
  - The VTC confirms that when the input is low, the output is high, and vice versa.
  - The switching region (where output transitions) is roughly around the mid-supply region (close to 0.7–0.9 V for a 1.8 V supply, depending on transistor sizing).

```markdown

```

- **Key learning points:**
  - How to create a custom library and attach a technology file in Cadence.
  - How to design a simple CMOS inverter schematic.
  - How to generate a symbol and then build a **test circuit** using that symbol.
  - How to set up and run both transient and DC simulations.
  - The importance of choosing realistic simulation stop times.


---

## 6. Video References

The implementation closely follows these YouTube videos for guidance on Cadence Virtuoso and inverter simulation:

1. Cadence Virtuoso – Inverter Design Intro  
   `https://www.youtube.com/watch?v=sOqgcOgouKk&list=PL86shurD5y344qr_c7b_o8jC3ucKfStmY`
2. Symbol and Test Circuit Construction  
   `https://www.youtube.com/watch?v=TcCHsO5j9CI&list=PL86shurD5y344qr_c7b_o8jC3ucKfStmY&index=2`
3. DC and Transient Analysis of Inverter  
   `https://www.youtube.com/watch?v=moXKwAzmxXU&list=PL86shurD5y344qr_c7b_o8jC3ucKfStmY&index=3`
4. Additional Inverter Flow Videos  
   `https://www.youtube.com/watch?v=4rMDzVLAAXo&list=PL86shurD5y344qr_c7b_o8jC3ucKfStmY&index=4`

---

## 7. Author

**Vansh Vaghela** – Electronics and Communication (VLSI) student, exploring custom IC design flows, Cadence Virtuoso, and digital/analog VLSI.
